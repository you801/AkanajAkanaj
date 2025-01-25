local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local ESPs = {}
local ESPEnabled = true
local AimbotEnabled = true
local NoRecoilEnabled = true
local FOVSize = 150
local AimSmoothness = 5
local AntiLagEnabled = false
local SkyRemoved = false
local PanelVisible = true

-- Função para remover texturas de objetos (armas e partes)
local function removeTexturesFromObject(object)
    -- Remove qualquer textura ou decal do objeto
    for _, descendant in pairs(object:GetDescendants()) do
        if descendant:IsA("Texture") or descendant:IsA("Decal") then
            descendant:Destroy()  -- Remover a textura ou decal
        end
    end
end

-- Função para remover texturas de armas do personagem
local function removeTexturesFromWeapons()
    -- Remover texturas de armas e ferramentas do personagem
    local character = LocalPlayer.Character
    if character then
        for _, tool in pairs(character:GetChildren()) do
            if tool:IsA("Tool") then
                removeTexturesFromObject(tool)
            end
        end
    end
end

-- Função para remover texturas de todos os objetos no jogo (exceto o personagem)
local function removeTexturesFromAllObjects()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Tool") then
            removeTexturesFromObject(obj)
        elseif obj:IsA("MeshPart") or obj:IsA("Part") then
            removeTexturesFromObject(obj)
        end
    end
end

-- Criar GUI do Painel Futurista
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 250, 0, 250)
Frame.Position = UDim2.new(0.05, 0, 0.2, 0)
Frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Frame.BackgroundTransparency = 0.2
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui

local function createToggle(yOffset, label, callback)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Size = UDim2.new(0, 50, 0, 25)
    toggleFrame.Position = UDim2.new(0.75, -25, 0, yOffset)
    toggleFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    toggleFrame.BorderSizePixel = 0
    toggleFrame.Parent = Frame

    local toggleButton = Instance.new("Frame")
    toggleButton.Size = UDim2.new(0, 20, 0, 20)
    toggleButton.Position = UDim2.new(0, 2, 0, 2)
    toggleButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    toggleButton.Parent = toggleFrame

    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(0, 100, 0, 25)
    textLabel.Position = UDim2.new(0, 10, 0, yOffset)
    textLabel.Text = label
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.BackgroundTransparency = 1
    textLabel.TextSize = 16
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.TextXAlignment = Enum.TextXAlignment.Left
    textLabel.Parent = Frame

    local isActive = false

    toggleFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isActive = not isActive
            if isActive then
                toggleButton:TweenPosition(UDim2.new(1, -22, 0, 2), "Out", "Sine", 0.2, true)
                toggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
            else
                toggleButton:TweenPosition(UDim2.new(0, 2, 0, 2), "Out", "Sine", 0.2, true)
                toggleButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
            end
            callback(isActive)
        end
    end)
end

createToggle(20, "ESP", function(state) ESPEnabled = state end)
createToggle(60, "Aimbot", function(state) AimbotEnabled = state end)
createToggle(100, "No Recoil", function(state) NoRecoilEnabled = state end)
createToggle(140, "FOV", function(state) FOVSize = state and 150 or 0 end)
createToggle(180, "Anti-Lag", function(state) 
    AntiLagEnabled = state
    if AntiLagEnabled then
        for _, obj in pairs(workspace:GetDescendants()) do
            if obj:IsA("Texture") or obj:IsA("Decal") then
                obj:Destroy()
            end
        end
        if not SkyRemoved and Lighting:FindFirstChildOfClass("Sky") then
            Lighting:FindFirstChildOfClass("Sky"):Destroy()
            SkyRemoved = true
        end
        Lighting.Ambient = Color3.new(0, 0, 0)
        removeTexturesFromAllObjects()  -- Remover texturas de todos os objetos
    end
end)

-- Toggle do painel com a tecla Insert
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.Insert then
        PanelVisible = not PanelVisible
        Frame.Visible = PanelVisible
    end
end)

-- Criar FOV visível
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

-- Criar ESP
local function CreateESP(player)
    if player == LocalPlayer or ESPs[player] then return end

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

    ESPs[player] = highlight
end

local function UpdateESP()
    if not ESPEnabled then return end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            if not ESPs[player] then
                CreateESP(player)
            end
        end
    end
end

-- Melhor Aimbot
local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = FOVSize

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local Head = player.Character:FindFirstChild("Head")
            if Head then
                local ScreenPosition, OnScreen = Camera:WorldToViewportPoint(Head.Position)

                if OnScreen then
                    local MousePos = UserInputService:GetMouseLocation()
                    local Distance = (Vector2.new(ScreenPosition.X, ScreenPosition.Y) - MousePos).Magnitude

                    if Distance < shortestDistance then
                        closestPlayer = Head.Position
                        shortestDistance = Distance
                    end
                end
            end
        end
    end
    return closestPlayer
end

RunService.RenderStepped:Connect(function()
    UpdateESP()

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
        local gun = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Tool")
        if gun and gun:FindFirstChild("Recoil") then
            gun.Recoil.Value = 0
        end
    end
end)
