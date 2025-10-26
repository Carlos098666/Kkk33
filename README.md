local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- UI Setup
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimAssistGui"
screenGui.Parent = player:WaitForChild("PlayerGui")

local aimButton = Instance.new("ImageButton")
aimButton.Size = UDim2.new(0, 80, 0, 80)
aimButton.Position = UDim2.new(0.5, -40, 0.8, 0)
aimButton.Image = "rbxassetid://3926305904" -- Troque para sua imagem!
aimButton.BackgroundTransparency = 0.4
aimButton.Parent = screenGui

-- Botão móvel
local dragging = false
local dragOffset = Vector2.new(0, 0)

aimButton.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragOffset = Vector2.new(input.Position.X, input.Position.Y) - Vector2.new(aimButton.AbsolutePosition.X, aimButton.AbsolutePosition.Y)
	end
end)
aimButton.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)
UserInputService.InputChanged:Connect(function(input)
	if dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
		local newPos = Vector2.new(input.Position.X, input.Position.Y) - dragOffset
		aimButton.Position = UDim2.new(0, newPos.X, 0, newPos.Y)
	end
end)

-- Função de checagem de time (adapte conforme a lógica de Mad City)
local function isEnemy(otherPlayer)
	-- Exemplo: Supondo que Mad City usa player.Team
	return otherPlayer.Team ~= player.Team
	-- Se Mad City usa outra lógica, troque a condição acima!
end

-- Aim Assist
local function getClosestEnemy()
	local char = player.Character
	if not char or not char:FindFirstChild("HumanoidRootPart") then return nil end

	local closestDist = math.huge
	local closestEnemy = nil

	for _, otherPlayer in ipairs(Players:GetPlayers()) do
		if otherPlayer ~= player and isEnemy(otherPlayer) then
			local otherChar = otherPlayer.Character
			if otherChar and otherChar:FindFirstChild("HumanoidRootPart") and otherChar:FindFirstChild("Humanoid") and otherChar.Humanoid.Health > 0 then
				local dist = (char.HumanoidRootPart.Position - otherChar.HumanoidRootPart.Position).Magnitude
				if dist < closestDist then
					closestDist = dist
					closestEnemy = otherChar
				end
			end
		end
	end

	return closestEnemy
end

local aiming = false

aimButton.MouseButton1Down:Connect(function()
	aiming = true
end)
aimButton.MouseButton1Up:Connect(function()
	aiming = false
end)
aimButton.TouchStarted:Connect(function()
	aiming = true
end)
aimButton.TouchEnded:Connect(function()
	aiming = false
end)

RunService.RenderStepped:Connect(function()
	if aiming then
		local enemy = getClosestEnemy()
		if enemy and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local cam = workspace.CurrentCamera
			cam.CFrame = CFrame.new(cam.CFrame.Position, enemy.HumanoidRootPart.Position)
		end
	end
end)
