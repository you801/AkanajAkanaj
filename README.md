local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Drawing = Drawing or {}

local ESPEnabled = true
local RageAimbotEnabled = false
local NoRecoilEnabled = true
local FOVEnabled = false
local FOVSize = 150
local ESPBoxSize = 50  -- Valor inicial da Box

-- Criar GUI do Painel Futurista
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 250, 0, 300)
Frame.Position = UDim2.new(0.05, 0, 0.2, 0)
Frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Frame.BackgroundTransparency = 0.2
Frame.BorderSizePixel = 0
Frame.Visible = true
Frame.Parent = ScreenGui

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
createToggle(60, "Rage Aimbot", function(state) RageAimbotEnabled = state end)
createToggle(100, "Sem Recuo", function(state) NoRecoilEnabled = state end)
createToggle(140, "FOV", function(state) FOVEnabled = state end)

-- Slider para ajustar o tamanho da Box ESP
local Slider = Instance.new("TextLabel")
Slider.Size = UDim2.new(0, 200, 0, 25)
Slider.Position = UDim2.new(0, 10, 0, 180)
Slider.Text = "Tamanho da Box ESP: " .. ESPBoxSize
Slider.TextColor3 = Color3.fromRGB(255, 255, 255)
Slider.BackgroundTransparency = 0.5
Slider.Font = Enum.Font.SourceSansBold
Slider.TextSize = 16
Slider.Parent = Frame

local function updateESPSize(value)
    ESPBoxSize = math.clamp(value, 1, 100) -- Garante que fique entre 1% e 100%
    Slider.Text = "Tamanho da Box ESP: " .. ESPBoxSize
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

-- Criar FOV Circle
local FOVCircle = Drawing.new("Circle")
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Thickness = 1
FOVCircle.NumSides = 50
FOVCircle.Radius = FOVSize
FOVCircle.Filled = false
FOVCircle.Visible = false

RunService.RenderStepped:Connect(function()
    if FOVEnabled then
        local MousePos = UserInputService:GetMouseLocation()
        FOVCircle.Position = MousePos
        FOVCircle.Radius = FOVSize
        FOVCircle.Visible = true
    else
        FOVCircle.Visible = false
    end
end)

-- Criar o gráfico de massinha (barra que aumenta)
local GraphBar = Instance.new("Frame")
GraphBar.Size = UDim2.new(0, 200, 0, 10)
GraphBar.Position = UDim2.new(0, 10, 0, 220)
GraphBar.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
GraphBar.BackgroundTransparency = 0.5
GraphBar.Parent = Frame
GraphBar.Visible = false

-- Função para mostrar gráfico de massinha
local function ShowGraphBar()
    GraphBar.Visible = true
    GraphBar:TweenSize(UDim2.new(0, 200, 0, 20), "Out", "Sine", 0.5, true)
end

local function HideGraphBar()
    GraphBar.Visible = false
end

-- Função para alternar o gráfico de massinha com o painel
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.Insert then
        if GraphBar.Visible then
            HideGraphBar()  -- Esconde o gráfico
        else
            ShowGraphBar()  -- Mostra o gráfico
        end
    end
end)

-- Melhor Rage Aimbot (Mira direto na cabeça)
local function GetClosestPlayer()
    local closestPlayer = nil

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            return player.Character.Head.Position
        end
    end
    return nil
end

RunService.RenderStepped:Connect(function()
    if RageAimbotEnabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local targetPos = GetClosestPlayer()
        if targetPos then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPos)
        end
    end
end)

-- Criar ESP Box
local ESPs = {}

local function CreateESP(player)
    if player == LocalPlayer or ESPs[player] then return end

    local box = Drawing.new("Square")
    box.Color = Color3.fromRGB(255, 0, 0)
    box.Thickness = 2
    box.Visible = false

    ESPs[player] = box

    player.CharacterAdded:Connect(function(character)
        box.Visible = true
    end)
end

local function UpdateESP()
    if not ESPEnabled then return end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if not ESPs[player] then
                CreateESP(player)
            end
            local box = ESPs[player]
            local position, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)

            if onScreen then
                box.Position = Vector2.new(position.X - (ESPBoxSize / 2), position.Y - (ESPBoxSize / 2))
                box.Size = Vector2.new(ESPBoxSize, ESPBoxSize)
                box.Visible = true
            else
                box.Visible = false
            end
        end
    end
end

-- Criar Ranger (a caixa azul de alcance)
local function CreateRanger(player)
    if player == LocalPlayer then return end
    
    local ranger = Drawing.new("Square")
    ranger.Color = Color3.fromRGB(0, 0, 255)  -- Azul
    ranger.Thickness = 2
    ranger.Visible = false
    ranger.Filled = false
    
    -- Atualiza a visibilidade da ranger
    player.CharacterAdded:Connect(function()
        ranger.Visible = true
    end)

    return ranger
end

-- Atualiza a posição e o tamanho da Ranger
local function UpdateRanger()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local position, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
            local ranger = CreateRanger(player)
            if onScreen then
                ranger.Position = Vector2.new(position.X - (ESPBoxSize / 2), position.Y - (ESPBoxSize / 2))
                ranger.Size = Vector2.new(ESPBoxSize * 2, ESPBoxSize * 2)  -- Ajuste no tamanho
                ranger.Visible = true
            else
                ranger.Visible = false
            end
        end
    end
end

RunService.RenderStepped:Connect(function()
    UpdateESP()
    UpdateRanger()
end)
