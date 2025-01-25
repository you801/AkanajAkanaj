Jogadores locais = jogo:GetService("Jogadores")
local RunService = jogo:GetService("RunService")
local UserInputService = jogo:GetService("UserInputService")
Iluminação local = jogo:GetService("Iluminação")
local LocalPlayer = Jogadores.LocalPlayer
Câmera local = workspace.CurrentCamera

ESPs locais = {}
local ESPEnabled = verdadeiro
AimbotEnabled local = verdadeiro
local NoRecoilEnabled = verdadeiro
tamanho do campo de visão local = 150
local AimSmoothness = 5
AntiLagEnabled local = falso
local SkyRemoved = falso
Painel localVisível = verdadeiro

-- Função para remover texturas de objetos (sem remover de armas)
função local removeTexturesFromObject(objeto)
    -- Remova qualquer textura ou decalque do objeto, exceto se for uma arma
    para _, descendente em pares(objeto:GetDescendants()) faça
        se descendente:IsA("Textura") ou descendente:IsA("Decalque") então
            se não descendente.Parent:IsA("Tool") then -- Não remove texturas de armas
                descendant:Destroy() -- Remove textura ou decalque
            fim
        fim
    fim
fim

-- Função para remover texturas de objetos no jogo, sem tocar nas armas
função local removeTexturesFromAllObjects()
    para _, obj em pares(workspace:GetDescendants()) faça
        se obj:IsA("Ferramenta") então
            -- Não remove texturas de ferramentas (armas)
        elseif obj:IsA("MeshPart") ou obj:IsA("Part") então
            removeTexturesFromObject(obj)
        fim
    fim
fim

-- Criar GUI do Painel Futurista
ScreenGui local = Instância.new("ScreenGui")
ScreenGui.Parent = jogo.CoreGui

Quadro local = Instance.new("Quadro")
Frame.Size = UDim2.new(0, 250, 0, 250)
Frame.Posição = UDim2.new(0,05, 0, 0,2, 0)
Quadro.BackgroundColor3 = Cor3.fromRGB(0, 0, 0)
Frame.BackgroundTransparência = 0,2
Quadro.BorderSizePixel = 0
Frame.Parent = TelaGuia

função local createToggle(yOffset, rótulo, retorno de chamada)
    local toggleFrame = Instance.new("Quadro")
    toggleFrame.Size = UDim2.new(0, 50, 0, 25)
    toggleFrame.Position = UDim2.new(0,75, -25, 0, yOffset)
    toggleFrame.BackgroundColor3 = Cor3.fromRGB(40, 40, 40)
    toggleFrame.BorderSizePixel = 0
    toggleFrame.Parent = Quadro

    local toggleButton = Instance.new("Quadro")
    toggleButton.Size = UDim2.new(0, 20, 0, 20)
    toggleButton.Posição = UDim2.novo(0, 2, 0, 2)
    toggleButton.BackgroundColor3 = Cor3.fromRGB(100, 100, 100)
    toggleButton.Parent = alternarQuadro

    textLabel local = Instância.new("TextLabel")
    textLabel.Size = UDim2.new(0, 100, 0, 25)
    textLabel.Posição = UDim2.new(0, 10, 0, yOffset)
    textLabel.Text = rótulo
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.BackgroundTransparência = 1
    textLabel.TamanhoDoTexto = 16
    textLabel.Font = Enum.Fonte.SourceSansBold
    textLabel.TextXAlignment = Enum.TextXAlignment.Esquerda
    textLabel.Parent = Quadro

    local isActive = falso

    toggleFrame.InputBegan:Connect(função(entrada)
        se input.UserInputType == Enum.UserInputType.MouseButton1 então
            isActive = não éAtivo
            se isActive então
                toggleButton:TweenPosition(UDim2.new(1, -22, 0, 2), "Saída", "Seno", 0.2, verdadeiro)
                toggleButton.BackgroundColor3 = Cor3.fromRGB(0, 255, 0)
            outro
                toggleButton:TweenPosition(UDim2.new(0, 2, 0, 2), "Fora", "Seno", 0.2, verdadeiro)
                toggleButton.BackgroundColor3 = Cor3.fromRGB(100, 100, 100)
            fim
            retorno de chamada(isActive)
        fim
    fim)
fim

createToggle(20, "ESP", função(estado) ESPEnabled = estado fim)
createToggle(60, "Aimbot", função(estado) AimbotEnabled = estado fim)
createToggle(100, "Sem recuo", function(state) NoRecoilEnabled = estado final)
createToggle(140, "FOV", função(estado) FOVSize = estado e 150 ou 0 fim)
createToggle(180, "Anti-Lag", função(estado)
    AntiLagEnabled = estado
    se AntiLagEnabled então
        para _, obj em pares(workspace:GetDescendants()) faça
            se obj:IsA("Textura") ou obj:IsA("Decalque") então
                obj:Destruir()
            fim
        fim
        se não SkyRemoved e Lighting:FindFirstChildOfClass("Sky") então
            Iluminação:FindFirstChildOfClass("Céu"):Destroy()
            SkyRemoved = verdadeiro
        fim
        Iluminação.Ambiente = Cor3.novo(0, 0, 0)
        removeTexturesFromAllObjects() -- Remove texturas de todos os objetos, exceto armas
    fim
fim)

-- Alternar do painel com a tecla Inserir
UserInputService.InputBegan:Connect(função(entrada, jogoProcessado)
    se não for gameProcessed e input.KeyCode == Enum.KeyCode.Insert então
        PanelVisible = não PanelVisible
        Frame.Visible = PainelVisível
    fim
fim)

-- Criar FOV visível
local FOVCircle = Desenho.new("Círculo")
FOVCircle.Color = Cor3.fromRGB(255, 255, 255)
FOVCircle.Espessura = 1
FOVCircle.NumSides = 50
FOVCircle.Radius = TamanhoFOV
FOVCircle.Filled = falso
FOVCircle.Visible = verdadeiro

RunService.RenderStepped:Connect(função()
    MousePos local = UserInputService:GetMouseLocation()
    FOVCircle.Posição = MousePos
    FOVCircle.Radius = TamanhoFOV
    FOVCircle.Visible = (FOVSize > 0)
fim)

-- Criar ESP
função local CreateESP(jogador)
    se jogador == LocalPlayer ou ESPs[jogador] então retorne fim

    destaque local = Instance.new("Destaque")
    destaque.FillColor = Cor3.fromRGB(255, 0, 0)
    destaque.OutlineColor = Cor3.fromRGB(255, 255, 255)
    destaque.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    
    player.CharacterAdded:Connect(função(char)
        destaque.Parent = char
    fim)
    
    se jogador.Personagem então
        destaque.Pai = jogador.Personagem
    fim

    ESPs[jogador] = destaque
fim

função local UpdateESP()
    se não ESPEnabled então retorne fim

    para _, jogador em pares(Players:GetPlayers()) faça
        se jogador ~= LocalPlayer e jogador.Character então
            se não ESPs[jogador] então
                CriarESP(jogador)
            fim
        fim
    fim
fim

-- Melhor Aimbot
função local GetClosestPlayer()
    jogadormaispróximo local = nulo
    Distância mais curta local = FOVSize

    para _, jogador em pares(Players:GetPlayers()) faça
        se jogador ~= LocalPlayer e jogador.Character e jogador.Character:FindFirstChild("HumanoidRootPart") então
            local Cabeça = jogador.Personagem:FindFirstChild("Cabeça")
            se Cabeça então
                Posição da tela local, Na tela = Câmera:WorldToViewportPoint(Cabeça.Posição)

                se OnScreen então
                    MousePos local = UserInputService:GetMouseLocation()
                    Distância local = (Vector2.new(ScreenPosition.X, ScreenPosition.Y) - MousePos).Magnitude

                    se Distância < shortestDistance então
                        Jogadormaispróximo = Cabeça.Posição
                        shortestDistance = Distância
                    fim
                fim
            fim
        fim
    fim
    retornar jogadormaispróximo
fim

RunService.RenderStepped:Connect(função()
    AtualizarESP()

    se AimbotEnabled e UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) então
        targetPos local = ObterJogadorMaisPróximo()
        se targetPos então
            local newCFrame = CFrame.new(Camera.CFrame.Position, targetPos)
            lerpSpeed ​​local = math.clamp(0,2 * Suavidade do Objetivo, 0,05, 0,7)
            Câmera.CFrame = Câmera.CFrame:Lerp(novoCFrame, lerpSpeed)
        fim
    fim
fim)

-- Sem recuo
RunService.RenderStepped:Connect(função()
    se NoRecoilEnabled então
        arma local = LocalPlayer.Character e LocalPlayer.Character:FindFirstChildWhichIsA("Ferramenta")
        se arma e arma:FindFirstChild("Recoil") então
            arma.Recoil.Value = 0
        fim
    fim
fim)
