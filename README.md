local UserInputService = game:GetService("UserInputService")
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 250, 0, 250)
Frame.Position = UDim2.new(0, 5, 0, 5)
Frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Frame.BackgroundTransparency = 0.2
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui
Frame.Visible = true  -- Inicialmente, o painel estará visível

-- Variável de controle de visibilidade
local PainelVisivel = true

-- Função para alternar visibilidade do painel com a tecla Insert
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.Insert then
        PainelVisivel = not PainelVisivel
        Frame.Visible = PainelVisivel
    end
end)

-- Intentions - Ajustar a visibilidade do painel com a tecla Intent
local function togglePanelVisibility(isVisible)
    Frame.Visible = isVisible
end

-- Função para mostrar a visibilidade do painel com Intenção
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed then
        if input.KeyCode == Enum.KeyCode.I then  -- Usando a tecla I para visibilidade
            togglePanelVisibility(true)  -- Torna o painel visível
        elseif input.KeyCode == Enum.KeyCode.O then  -- Usando a tecla O para ocultar
            togglePanelVisibility(false)  -- Torna o painel invisível
        end
    end
end)

-- Aqui você pode adicionar outras funções para interações, como controles de ESP, Aimbot, etc.

-- Exemplo de Toggle de ESP
local ESPEnabled = false
local function toggleESP(state)
    ESPEnabled = state
    print("ESP Ativado: " .. tostring(state))
end

-- Criar toggle de ESP no painel
local function createESPButton(yOffset)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Size = UDim2.new(0, 50, 0, 25)
    toggleFrame.Position = UDim2.new(0, 75, 0, yOffset)
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
    textLabel.Text = "ESP"
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
            toggleESP(isActive)
        end
    end)
end

-- Criar botão de ESP no painel
createESPButton(20)
