local library = loadstring(game:HttpGet("https://raw.githubusercontent.com/bloodball/-back-ups-for-libs/main/Splix"))()

local window = library:new({textsize = 13.5,font = Enum.Font.RobotoMono,name = "Spectrum 11 | by DISHONORABLE",color = Color3.fromRGB(225,58,81)})

local tab = window:page({name = "Player"})

	local WS = tab:section({name = "WalkSpeed",side = "left",size = 90})

		WS:textbox({name = "Speed",def = "Enter WalkSpeed",placeholder = "Enter WalkSpeed",callback = function(value)
  			 game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = (value)
		end})

		WS:button({name = "Reset WalkSpeed",callback = function()
   			game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = 16
		end})

	local JP = tab:section({name = "JumpHeight",side = "right",size = 90})

		JP:textbox({name = "Height",def = "Enter JumpHeight",placeholder = "Enter JumpHeight",callback = function(value)
  			 game.Players.LocalPlayer.Character.Humanoid.JumpPower = (value)
		end})

		JP:button({name = "Reset JumpPower",callback = function()
   			game.Players.LocalPlayer.Character.Humanoid.JumpPower = 50
		end})

local VIS = window:page({name = "Visuals"})

	local ESP = VIS:section({name = "ESP",side = "left",size = 90})

		ESP:toggle({name = "Boxes",def = false,callback = function(value)
			tog = value
			print("ESP Toggled: " .. tostring(tog))
	
			-- settings
			local settings = {
				defaultcolor = Color3.fromRGB(255,0,0),
				teamcheck = false,
				teamcolor = true
			}
	
			-- services
			local runService = game:GetService("RunService")
			local players = game:GetService("Players")
	
			-- variables
			local localPlayer = players.LocalPlayer
			local camera = workspace.CurrentCamera
			local newVector2, newColor3, newDrawing = Vector2.new, Color3.new, Drawing.new
			local tan, rad = math.tan, math.rad
			local round = function(...) local a = {}; for i,v in next, table.pack(...) do a[i] = math.round(v); end return unpack(a); end
			local wtvp = function(...) local a, b = camera.WorldToViewportPoint(camera, ...) return newVector2(a.X, a.Y), b, a.Z end
	
			-- reuse storage across toggle
			if not shared.espCache then
				shared.espCache = {}
				shared.connections = {}
			end
	
			local espCache = shared.espCache
			local connections = shared.connections
	
			local function createEsp(player)
				local drawings = {}
				drawings.box = newDrawing("Square")
				drawings.box.Thickness = 1
				drawings.box.Filled = false
				drawings.box.Color = settings.defaultcolor
				drawings.box.Visible = false
				drawings.box.ZIndex = 2
	
				drawings.boxoutline = newDrawing("Square")
				drawings.boxoutline.Thickness = 3
				drawings.boxoutline.Filled = false
				drawings.boxoutline.Color = newColor3()
				drawings.boxoutline.Visible = false
				drawings.boxoutline.ZIndex = 1
	
				espCache[player] = drawings
			end
	
			local function removeEsp(player)
				if rawget(espCache, player) then
					for _, drawing in next, espCache[player] do
						drawing:Remove()
					end
					espCache[player] = nil
				end
			end
	
			local function updateEsp(player, esp)
				local character = player and player.Character
				if character then
					local cframe = character:GetModelCFrame()
					local position, visible, depth = wtvp(cframe.Position)
					esp.box.Visible = visible
					esp.boxoutline.Visible = visible
	
					if cframe and visible then
						local scaleFactor = 1 / (depth * tan(rad(camera.FieldOfView / 2)) * 2) * 1000
						local width, height = round(4 * scaleFactor, 5 * scaleFactor)
						local x, y = round(position.X, position.Y)
	
						esp.box.Size = newVector2(width, height)
						esp.box.Position = newVector2(round(x - width / 2, y - height / 2))
						esp.box.Color = settings.teamcolor and player.TeamColor.Color or settings.defaultcolor
	
						esp.boxoutline.Size = esp.box.Size
						esp.boxoutline.Position = esp.box.Position
					end
				else
					esp.box.Visible = false
					esp.boxoutline.Visible = false
				end
			end
	
			if tog then
				-- create for current players
				for _, player in next, players:GetPlayers() do
					if player ~= localPlayer and not espCache[player] then
						createEsp(player)
					end
				end
	
				-- bind player events once
				if not connections.PlayerAdded then
					connections.PlayerAdded = players.PlayerAdded:Connect(function(player)
						if tog then createEsp(player) end
					end)
				end
	
				if not connections.PlayerRemoving then
					connections.PlayerRemoving = players.PlayerRemoving:Connect(function(player)
						removeEsp(player)
					end)
				end
	
				-- bind draw loop
				if not connections.Render then
					runService:BindToRenderStep("esp", Enum.RenderPriority.Camera.Value, function()
						for player, drawings in next, espCache do
							if settings.teamcheck and player.Team == localPlayer.Team then
								drawings.box.Visible = false
								drawings.boxoutline.Visible = false
							elseif player ~= localPlayer then
								updateEsp(player, drawings)
							end
						end
					end)
					connections.Render = true
				end
	
			else
				-- disable all drawings
				for player, drawings in next, espCache do
					if drawings then
						for _, d in pairs(drawings) do
							d.Visible = false
							d:Remove()
						end
					end
				end
				table.clear(espCache)
	
				-- disconnect events
				if connections.PlayerAdded then connections.PlayerAdded:Disconnect() end
				if connections.PlayerRemoving then connections.PlayerRemoving:Disconnect() end
				connections.PlayerAdded = nil
				connections.PlayerRemoving = nil
	
				-- unbind draw loop
				if connections.Render then
					runService:UnbindFromRenderStep("esp")
					connections.Render = nil
				end
			end
		end
	})

	ESP:toggle({name = "Tracers",def = false,callback = function(value)
        tog = value
        print("Tracers Toggled: " .. tostring(tog))

        -- settings
        local settings = {
            tracercolor = Color3.fromRGB(0, 255, 0),
            teamcheck = false,
            teamcolor = true,
            origin = "Bottom" -- "Bottom" or "Center"
        }

        -- services
        local runService = game:GetService("RunService")
        local players = game:GetService("Players")

        -- variables
        local localPlayer = players.LocalPlayer
        local camera = workspace.CurrentCamera
        local newVector2, newColor3, newDrawing = Vector2.new, Color3.new, Drawing.new
        local wtvp = function(...) local a, b = camera.WorldToViewportPoint(camera, ...) return newVector2(a.X, a.Y), b, a.Z end

        -- shared table to persist between toggles
        if not shared.tracerCache then
            shared.tracerCache = {}
            shared.tracerConnections = {}
        end

        local tracerCache = shared.tracerCache
        local connections = shared.tracerConnections

        local function createTracer(player)
            local line = newDrawing("Line")
            line.Thickness = 1.5
            line.Color = settings.tracercolor
            line.Visible = false
            line.ZIndex = 2

            tracerCache[player] = line
        end

        local function removeTracer(player)
            if rawget(tracerCache, player) then
                tracerCache[player]:Remove()
                tracerCache[player] = nil
            end
        end

        local function updateTracer(player, line)
            local character = player and player.Character
            if character then
                local cframe = character:GetModelCFrame()
                local screenPos, onScreen = wtvp(cframe.Position)

                line.Visible = onScreen

                if onScreen then
                    local originPoint
                    if settings.origin == "Center" then
                        originPoint = newVector2(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
                    else
                        originPoint = newVector2(camera.ViewportSize.X / 2, camera.ViewportSize.Y)
                    end

                    line.From = originPoint
                    line.To = screenPos
                    line.Color = settings.teamcolor and player.TeamColor.Color or settings.tracercolor
                end
            else
                line.Visible = false
            end
        end

        if tog then
            -- create tracers for all players
            for _, player in next, players:GetPlayers() do
                if player ~= localPlayer and not tracerCache[player] then
                    createTracer(player)
                end
            end

            -- listen for new players
            if not connections.PlayerAdded then
                connections.PlayerAdded = players.PlayerAdded:Connect(function(player)
                    if tog then createTracer(player) end
                end)
            end

            -- listen for players leaving
            if not connections.PlayerRemoving then
                connections.PlayerRemoving = players.PlayerRemoving:Connect(function(player)
                    removeTracer(player)
                end)
            end

            -- bind drawing loop
            if not connections.Render then
                runService:BindToRenderStep("tracers", Enum.RenderPriority.Camera.Value, function()
                    for player, line in next, tracerCache do
                        if settings.teamcheck and player.Team == localPlayer.Team then
                            line.Visible = false
                        elseif player ~= localPlayer then
                            updateTracer(player, line)
                        end
                    end
                end)
                connections.Render = true
            end

        else
            -- hide and remove all tracers
            for player, line in next, tracerCache do
                if line then
                    line.Visible = false
                    line:Remove()
                end
            end
            table.clear(tracerCache)

            -- disconnect signals
            if connections.PlayerAdded then connections.PlayerAdded:Disconnect() end
            if connections.PlayerRemoving then connections.PlayerRemoving:Disconnect() end
            connections.PlayerAdded = nil
            connections.PlayerRemoving = nil

            -- unbind draw loop
            if connections.Render then
                runService:UnbindFromRenderStep("tracers")
                connections.Render = nil
            end
        end
    end
})

	ESP:toggle({name = "Username",def = false,callback = function(value)
        tog = value
        print("Usernames Toggled: " .. tostring(tog))

        -- settings
        local settings = {
            textcolor = Color3.fromRGB(255, 255, 255),
            teamcheck = false,
            teamcolor = true
        }

        -- services
        local runService = game:GetService("RunService")
        local players = game:GetService("Players")

        -- variables
        local localPlayer = players.LocalPlayer
        local camera = workspace.CurrentCamera
        local newVector2, newColor3, newDrawing = Vector2.new, Color3.new, Drawing.new
        local wtvp = function(...) local a, b = camera.WorldToViewportPoint(camera, ...) return newVector2(a.X, a.Y), b, a.Z end

        -- shared table for usernames
        if not shared.usernameCache then
            shared.usernameCache = {}
            shared.usernameConnections = {}
        end

        local usernameCache = shared.usernameCache
        local connections = shared.usernameConnections

        local function createUsernameEsp(player)
            local text = newDrawing("Text")
            text.Size = 13
            text.Color = settings.textcolor
            text.Center = true
            text.Outline = true
            text.Text = player.Name
            text.Visible = false
            text.ZIndex = 2

            usernameCache[player] = text
        end

        local function removeUsernameEsp(player)
            if rawget(usernameCache, player) then
                usernameCache[player]:Remove()
                usernameCache[player] = nil
            end
        end

        local function updateUsernameEsp(player, textObj)
            local character = player and player.Character
            if character and character:FindFirstChild("Head") then
                local headPos = character.Head.Position + Vector3.new(0, 2, 0) -- above head
                local screenPos, onScreen = wtvp(headPos)

                textObj.Visible = onScreen

                if onScreen then
                    textObj.Position = screenPos
                    textObj.Text = player.Name
                    textObj.Color = settings.teamcolor and player.TeamColor.Color or settings.textcolor
                end
            else
                textObj.Visible = false
            end
        end

        if tog then
            for _, player in next, players:GetPlayers() do
                if player ~= localPlayer and not usernameCache[player] then
                    createUsernameEsp(player)
                end
            end

            if not connections.PlayerAdded then
                connections.PlayerAdded = players.PlayerAdded:Connect(function(player)
                    if tog then createUsernameEsp(player) end
                end)
            end

            if not connections.PlayerRemoving then
                connections.PlayerRemoving = players.PlayerRemoving:Connect(function(player)
                    removeUsernameEsp(player)
                end)
            end

            if not connections.Render then
                runService:BindToRenderStep("usernames", Enum.RenderPriority.Camera.Value, function()
                    for player, textObj in next, usernameCache do
                        if settings.teamcheck and player.Team == localPlayer.Team then
                            textObj.Visible = false
                        elseif player ~= localPlayer then
                            updateUsernameEsp(player, textObj)
                        end
                    end
                end)
                connections.Render = true
            end

        else
            -- Hide and remove all username ESPs
            for player, textObj in next, usernameCache do
                if textObj then
                    textObj.Visible = false
                    textObj:Remove()
                end
            end
            table.clear(usernameCache)

            -- Disconnect events
            if connections.PlayerAdded then connections.PlayerAdded:Disconnect() end
            if connections.PlayerRemoving then connections.PlayerRemoving:Disconnect() end
            connections.PlayerAdded = nil
            connections.PlayerRemoving = nil

            -- Unbind render loop
            if connections.Render then
                runService:UnbindFromRenderStep("usernames")
                connections.Render = nil
            end
        end
    end
})

local aim = VIS:section({name = "Aimbot",side = "right",size = 100})
	
aim:toggle({name = "Toggle",def = false,callback = function(value)
	tog = value
    print("Aimbot Toggled: " .. tostring(tog))

    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local UserInputService = game:GetService("UserInputService")

    local player = Players.LocalPlayer
    local camera = workspace.CurrentCamera

    -- Store these globally for reuse between toggles
    if not shared.aimConnection then
        shared.aimConnection = nil
    end
    if not shared.fovCircle then
        shared.fovCircle = nil
    end

    local function getClosestTarget()
        local closest = nil
        local shortestDistance = 150

        for _, target in ipairs(Players:GetPlayers()) do
            if target ~= player and target.Character and target.Character:FindFirstChild("Head") then
                local head = target.Character.Head
                local worldPos = head.Position
                local screenPos, onScreen = camera:WorldToViewportPoint(worldPos)
                if onScreen then
                    local mousePos = UserInputService:GetMouseLocation()
                    local distance = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    if distance < shortestDistance then
                        shortestDistance = distance
                        closest = target
                    end
                end
            end
        end
        return closest
    end

    if tog then
        -- Create FOV Circle
        shared.fovCircle = Drawing.new("Circle")
        shared.fovCircle.Visible = true
        shared.fovCircle.Color = Color3.fromRGB(255, 255, 255)
        shared.fovCircle.Thickness = 1
        shared.fovCircle.Filled = false
        shared.fovCircle.Radius = 100

        -- Start Aimbot Logic
        shared.aimConnection = RunService.RenderStepped:Connect(function()
            local mousePos = UserInputService:GetMouseLocation()
            shared.fovCircle.Position = mousePos

            if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
                local target = getClosestTarget()
                if target and target.Character and target.Character:FindFirstChild("Head") then
                    local headPos = target.Character.Head.Position
                    camera.CFrame = camera.CFrame:Lerp(CFrame.new(camera.CFrame.Position, headPos), 0.2)
                end
            end
        end)

    else
        -- Clean Up
        if shared.aimConnection then
            shared.aimConnection:Disconnect()
            shared.aimConnection = nil
        end
        if shared.fovCircle then
            shared.fovCircle:Remove()
            shared.fovCircle = nil
        end
    end
end
})



local MISC = window:page({name = "Misc"})

	local ADMIN = MISC:section({name = "Commands",side = "left",size = 50})

		ADMIN:button({name = "Infinite Yield",callback = function()
   			loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
		end})

	local VC_BAN = MISC:section({name = "Remove VC-Ban",side = "Right",size = 50})

		VC_BAN:button({name = "UnBan",callback = function()
   			local Hint = Instance.new("Hint", workspace)
			Hint.Text = "You've been successfully UnBanned!"
			game:GetService("VoiceChatService"):joinVoice()
			wait(3)
			Hint:Destroy()
		end})

	local jorking = MISC:section({name = "Jerk Off",side = "left",size = 75})

	jorking:button({name = "R15",callback = function()
		loadstring(game:HttpGet("https://pastefy.app/YZoglOyJ/raw"))()
 	end})

	jorking:button({name = "R6",callback = function()
		loadstring(game:HttpGet("https://pastefy.app/wa3v2Vgm/raw"))()
 	end})

local section1 = tab:section({name = "section1",side = "left",size = 250})

local multisection = tab:multisection({name = "multisection",side = "right",size = 250})

local section2 = multisection:section({name = "section2",side = "right",size = 100})

section1:toggle({name = "toggle",def = false,callback = function(value)
  tog = value
	print(tog)
end})

section1:button({name = "button",callback = function()
   print('hot ui lib')
end})

section1:slider({name = "rate ui lib 1-100",def = 1, max = 100,min = 1,rounding = true,ticking = false,measuring = "",callback = function(value)
   print(value)
end})

section1:dropdown({name = "dropdown",def = "",max = 10,options = {"1","2","3","4","5","6","7","8","9","10"},callback = function(chosen)
   print(chosen)
end})
-- for dropdowns put max to the number of items u have in the opions

section1:buttonbox({name = "buttonbox",def = "",max = 4, options = {"yoyoyo","yo2","yo3","yo4"},callback = function(value)
   print(value)
end})	


section1:multibox({name = "multibox",def = {}, max = 4,options = {"1","2","3","4"},callback = function(value)
   print(value)
end})

section1:textbox({name = "textbox",def = "default text",placeholder = "Enter WalkSpeed",callback = function(value)
   print(value)
end})

section1:keybind({name = "set ui keybind",def = nil,callback = function(key)
   window.key = key
end})

local picker = section1:colorpicker({name = "color",cpname = nil,def = Color3.fromRGB(255,255,255),callback = function(value)
   color = value
end})
