local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local ESPEnabled = true
local AimbotEnabled = true
local NoRecoilEnabled = true
local FOVSize = 150
local AimSmoothness = 5
local PanelVisible = true

-- Criar GUI do Painel
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 250, 0, 250)
Frame.Position = UDim2.new(0.05, 0, 0.2, 0)
Frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Frame.BackgroundTransparency = 0.2
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui

-- Criar botão de alternância (Toggle)
local function createToggle(yOffset, label, callback)
    local toggleButton = Instance.new("TextButton")
    toggleButton.Size = UDim2.new(0, 100, 0, 25)
    toggleButton.Position = UDim2.new(0.1, 0, 0, yOffset)
    toggleButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    toggleButton.Text = label
    toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleButton.Parent = Frame

    local isActive = false

    toggleButton.MouseButton1Click:Connect(function()
        isActive = not isActive
        if isActive then
            toggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        else
            toggleButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        end
        callback(isActive)
    end)
end

createToggle(20, "ESP", function(state) ESPEnabled = state end)
createToggle(60, "Aimbot", function(state) AimbotEnabled = state end)
createToggle(100, "No Recoil", function(state) NoRecoilEnabled = state end)
createToggle(140, "FOV", function(state) FOVSize = state and 150 or 0 end)

-- Alternar painel com tecla Insert
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.Insert then
        PanelVisible = not PanelVisible
        Frame.Visible = PanelVisible
    end
end)

-- Criar Círculo de FOV
local FOVCircle = Drawing.new("Circle")
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Thickness = 1
FOVCircle.NumSides = 50
FOVCircle.Radius = FOVSize
FOVCircle.Filled = false
FOVCircle.Visible = true

RunService.RenderStepped:Connect(function()
    local MousePos = UserInputService:GetMouseLocation()
    FOVCircle.Position = MousePos
    FOVCircle.Radius = FOVSize
    FOVCircle.Visible = (FOVSize > 0)
end)

-- Criar ESP para jogadores
local function CreateESP(player)
    if player == LocalPlayer then return end

    local highlight = Instance.new("Highlight")
    highlight.FillColor = Color3.fromRGB(255, 0, 0)
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop

    player.CharacterAdded:Connect(function(char)
        highlight.Parent = char
    end)

    if player.Character then
        highlight.Parent = player.Character
    end
end

-- Ativar ESP para todos os jogadores
local function UpdateESP()
    if not ESPEnabled then return end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            CreateESP(player)
        end
    end
end

RunService.RenderStepped:Connect(UpdateESP)

-- Aimbot
local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = FOVSize

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local Head = player.Character.Head
            local screenPosition, onScreen = Camera:WorldToViewportPoint(Head.Position)

            if onScreen then
                local mousePos = UserInputService:GetMouseLocation()
                local distance = (Vector2.new(screenPosition.X, screenPosition.Y) - mousePos).Magnitude

                if distance < shortestDistance then
                    closestPlayer = Head.Position
                    shortestDistance = distance
                end
            end
        end
    end
    return closestPlayer
end

RunService.RenderStepped:Connect(function()
    if AimbotEnabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local targetPos = GetClosestPlayer()
        if targetPos then
            local newCFrame = CFrame.new(Camera.CFrame.Position, targetPos)
            local lerpSpeed = math.clamp(0.2 * AimSmoothness, 0.05, 0.7)
            Camera.CFrame = Camera.CFrame:Lerp(newCFrame, lerpSpeed)
        end
    end
end)

-- No Recoil
RunService.RenderStepped:Connect(function()
    if NoRecoilEnabled then
        local weapon = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Tool")
        if weapon and weapon:FindFirstChild("Recoil") then
            weapon.Recoil.Value = 0
        end
    end
end)
