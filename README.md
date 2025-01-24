local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Drawing = Drawing or {}

local ESPEnabled = true
local RangerEnabled = true
local ESPBoxSize = 50  -- Valor inicial da Box
local RangerSize = ESPBoxSize + 20  -- Ranger maior que a ESP

-- Criar GUI do Painel Futurista
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 250, 0, 350)
Frame.Position = UDim2.new(0.05, 0, 0.2, 0)
Frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Frame.BackgroundTransparency = 0.2
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui

-- Botão para Fechar o Painel
local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 25, 0, 25)
CloseButton.Position = UDim2.new(1, -30, 0, 5)
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
CloseButton.Parent = Frame

CloseButton.MouseButton1Click:Connect(function()
    Frame.Visible = false
end)

-- Função para criar toggle no painel
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
                toggleButton.Position = UDim2.new(1, -22, 0, 2)
                toggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
            else
                toggleButton.Position = UDim2.new(0, 2, 0, 2)
                toggleButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
            end
            callback(isActive)
        end
    end)
end

createToggle(20, "ESP", function(state) ESPEnabled = state end)
createToggle(60, "Ranger", function(state) RangerEnabled = state end)

-- Criar Slider para ajustar tamanho da ESP Box
local Slider = Instance.new("TextLabel")
Slider.Size = UDim2.new(0, 200, 0, 25)
Slider.Position = UDim2.new(0, 10, 0, 100)
Slider.Text = "Tamanho da ESP Box: " .. ESPBoxSize
Slider.TextColor3 = Color3.fromRGB(255, 255, 255)
Slider.BackgroundTransparency = 0.5
Slider.Font = Enum.Font.SourceSansBold
Slider.TextSize = 16
Slider.Parent = Frame

local function updateESPSize(value)
    ESPBoxSize = math.clamp(value, 10, 150)  -- Garante que fique entre 10 e 150
    RangerSize = ESPBoxSize + 20
    Slider.Text = "Tamanho da ESP Box: " .. ESPBoxSize
end

-- Controle de ajuste via teclado
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed then
        if input.KeyCode == Enum.KeyCode.Left then
            updateESPSize(ESPBoxSize - 5)  -- Diminuir tamanho
        elseif input.KeyCode == Enum.KeyCode.Right then
            updateESPSize(ESPBoxSize + 5)  -- Aumentar tamanho
        end
    end
end)

-- Criar ESP e Ranger
local ESPs = {}
local Rangers = {}

local function CreateESP(player)
    if player == LocalPlayer or ESPs[player] then return end

    local box = Drawing.new("Square")
    box.Color = Color3.fromRGB(255, 0, 0)  -- Vermelho
    box.Thickness = 2
    box.Visible = false

    local ranger = Drawing.new("Square")
    ranger.Color = Color3.fromRGB(0, 0, 255)  -- Azul
    ranger.Thickness = 1
    ranger.Visible = false
    ranger.Transparency = 0.5  -- Transparente

    ESPs[player] = box
    Rangers[player] = ranger

    player.CharacterAdded:Connect(function(character)
        box.Visible = true
        ranger.Visible = true
    end)
end

local function UpdateESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if not ESPs[player] then
                CreateESP(player)
            end
            local box = ESPs[player]
            local ranger = Rangers[player]
            local position, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)

            if onScreen then
                box.Position = Vector2.new(position.X - (ESPBoxSize / 2), position.Y - (ESPBoxSize / 2))
                box.Size = Vector2.new(ESPBoxSize, ESPBoxSize)
                box.Visible = ESPEnabled

                ranger.Position = Vector2.new(position.X - (RangerSize / 2), position.Y - (RangerSize / 2))
                ranger.Size = Vector2.new(RangerSize, RangerSize)
                ranger.Visible = RangerEnabled
            else
                box.Visible = false
                ranger.Visible = false
            end
        end
    end
end

RunService.RenderStepped:Connect(UpdateESP)
