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
local AntiLagEnabled = false
local PanelVisible = true

-- Criar Círculo de FOV usando Drawing API
local FOVCircle = Drawing.new("Circle")
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Thickness = 1
FOVCircle.NumSides = 50
FOVCircle.Radius = FOVSize
FOVCircle.Filled = false
FOVCircle.Visible = true

-- Atualizar posição do círculo de FOV
RunService.RenderStepped:Connect(function()
    local MousePos = UserInputService:GetMouseLocation()
    FOVCircle.Position = MousePos
    FOVCircle.Radius = FOVSize
    FOVCircle.Visible = (FOVSize > 0)
end)

-- Melhor Aimbot
local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = FOVSize

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local Head = player.Character:FindFirstChild("Head")
            if Head then
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
    end
    return closestPlayer
end

-- Aplicar Aimbot ao jogador mais próximo
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
