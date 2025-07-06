local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local AimFOV = 100
local AimbotOn = false

local fovCircle = Drawing.new("Circle")
fovCircle.Color = Color3.fromRGB(255,0,0)
fovCircle.Thickness = 2
fovCircle.Filled = false
fovCircle.Visible = false

RunService.RenderStepped:Connect(function()
    fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    fovCircle.Radius = AimFOV
    fovCircle.Visible = AimbotOn
end)

local function GetClosestPlayer()
    local closest = nil
    local shortestDist = AimFOV
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
            local headPos, onScreen = Camera:WorldToViewportPoint(plr.Character.Head.Position)
            if onScreen then
                local screenPos = Vector2.new(headPos.X, headPos.Y)
                local dist = (screenPos - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                if dist < shortestDist then
                    shortestDist = dist
                    closest = plr
                end
            end
        end
    end
    return closest
end

RunService.RenderStepped:Connect(function()
    if AimbotOn then
        local target = GetClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.Head.Position)
        end
    end
end)

local gui = Instance.new("ScreenGui")
gui.Name = "BerserkHub"
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 220, 0, 160)
frame.Position = UDim2.new(0.5, -110, 0.5, -80)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(200, 0, 0)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.Text = "Berserk Hub"
title.Parent = frame

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 200, 0, 30)
toggleBtn.Position = UDim2.new(0.5, -100, 0, 35)
toggleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
toggleBtn.TextColor3 = Color3.new(1,1,1)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 16
toggleBtn.Text = "Ativar Aimbot"
toggleBtn.Parent = frame

toggleBtn.MouseButton1Click:Connect(function()
    AimbotOn = not AimbotOn
    toggleBtn.Text = AimbotOn and "Desativar Aimbot" or "Ativar Aimbot"
    toggleBtn.BackgroundColor3 = AimbotOn and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(60, 60, 60)
end)

local fovLabel = Instance.new("TextLabel")
fovLabel.Size = UDim2.new(0, 200, 0, 25)
fovLabel.Position = UDim2.new(0.5, -100, 0, 75)
fovLabel.BackgroundTransparency = 1
fovLabel.TextColor3 = Color3.new(1,1,1)
fovLabel.Font = Enum.Font.Gotham
fovLabel.TextSize = 16
fovLabel.Text = "FOV: "..AimFOV
fovLabel.Parent = frame

local plusBtn = Instance.new("TextButton")
plusBtn.Size = UDim2.new(0, 40, 0, 25)
plusBtn.Position = UDim2.new(0, 130, 0, 110)
plusBtn.Text = "+"
plusBtn.Font = Enum.Font.GothamBold
plusBtn.TextSize = 20
plusBtn.TextColor3 = Color3.new(1,1,1)
plusBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
plusBtn.Parent = frame

local minusBtn = Instance.new("TextButton")
minusBtn.Size = UDim2.new(0, 40, 0, 25)
minusBtn.Position = UDim2.new(0, 50, 0, 110)
minusBtn.Text = "-"
minusBtn.Font = Enum.Font.GothamBold
minusBtn.TextSize = 20
minusBtn.TextColor3 = Color3.new(1,1,1)
minusBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
minusBtn.Parent = frame

plusBtn.MouseButton1Click:Connect(function()
    AimFOV += 10
    fovLabel.Text = "FOV: "..AimFOV
end)

minusBtn.MouseButton1Click:Connect(function()
    AimFOV = math.max(10, AimFOV - 10)
    fovLabel.Text = "FOV: "..AimFOV
end)

local dragging, dragInput, dragStart, startPos

frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

frame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input == dragInput then
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X,
                                   startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
