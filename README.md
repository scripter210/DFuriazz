local player = game.Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local screenGui = nil  -- Variável para armazenar o ScreenGui
local painelAberto = true  -- Inicia o painel como aberto

-- Função para criar a GUI inicial
function criarTelaDflay()
    -- Se já existe o screenGui, não criamos outro
    if screenGui then return end

    -- Criando a ScreenGui
    screenGui = Instance.new("ScreenGui")
    screenGui.Name = "Dflay"
    screenGui.Parent = playerGui

    -- Criando o Frame (menu principal) 30% menor e depois mais 10% (total de 25%)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0.25, 0, 0.25, 0)  -- Reduzido para 25% da tela
    frame.Position = UDim2.new(0.375, 0, 0.375, 0)  -- Recentrado
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    frame.BorderSizePixel = 0
    frame.ClipsDescendants = true
    frame.Parent = screenGui

    -- Bordas arredondadas
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0.1, 0)
    corner.Parent = frame

    -- Título "Dflay"
    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, 0, 0.25, 0)
    textLabel.Position = UDim2.new(0, 0, 0, 0)
    textLabel.Text = "Dflay"
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.TextScaled = true
    textLabel.BackgroundTransparency = 1
    textLabel.Parent = frame

    -- Campo de senha
    local passwordBox = Instance.new("TextBox")
    passwordBox.Size = UDim2.new(0.8, 0, 0.25, 0)
    passwordBox.Position = UDim2.new(0.1, 0, 0.35, 0)
    passwordBox.PlaceholderText = "Digite a senha"
    passwordBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    passwordBox.TextSize = 20
    passwordBox.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    passwordBox.ClearTextOnFocus = true
    passwordBox.Parent = frame

    -- Função para verificar a senha
    passwordBox.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            -- Remove mensagens anteriores (se existirem)
            for _, child in pairs(frame:GetChildren()) do
                if child:IsA("TextLabel") and child.Name == "Feedback" then
                    child:Destroy()
                end
            end

            -- Mensagem de feedback
            local feedbackLabel = Instance.new("TextLabel")
            feedbackLabel.Name = "Feedback"
            feedbackLabel.Size = UDim2.new(1, 0, 0.2, 0)
            feedbackLabel.Position = UDim2.new(0, 0, 0.7, 0)
            feedbackLabel.BackgroundTransparency = 1
            feedbackLabel.TextScaled = true
            feedbackLabel.TextSize = 20
            feedbackLabel.Parent = frame

            if passwordBox.Text == "123456" then
                -- Senha correta: remover o painel de senha e mostrar apenas o título e o botão de Highlight
                passwordBox:Destroy()  -- Remove o campo de senha
                feedbackLabel:Destroy()  -- Remove a mensagem de feedback

                -- Reduzir o tamanho do título "Dflay"
                textLabel.TextSize = 18  -- Fazendo o título "Dflay" menor

                -- Redimensionar o painel para 50% da tela
                frame.Size = UDim2.new(0.5, 0, 0.5, 0)  -- Aumentado para 50% da tela
                frame.Position = UDim2.new(0.25, 0, 0.25, 0)  -- Reposiciona o painel

                -- Criar o botão de Highlight
                criarBotaoHighlight(frame)
            else
                feedbackLabel.Text = "Senha incorreta. Tente novamente."
                feedbackLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
            end
        end
    end)
end

-- Função para criar o botão de Highlight
function criarBotaoHighlight(parentFrame)
    -- Variável para controlar o estado do Highlight
    local highlightAtivado = false

    -- Criando o botão de Highlight
    local highlightButton = Instance.new("TextButton")
    highlightButton.Size = UDim2.new(0.8, 0, 0.1, 0)
    highlightButton.Position = UDim2.new(0.1, 0, 0.3, 0)
    highlightButton.Text = "[Highlight] OFF"
    highlightButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    highlightButton.TextSize = 20
    highlightButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    highlightButton.Parent = parentFrame

    -- Função associada ao botão de Highlight
    highlightButton.MouseButton1Click:Connect(function()
        highlightAtivado = not highlightAtivado  -- Alterna o estado do Highlight

        if highlightAtivado then
            highlightButton.Text = "[Highlight] ON"
            print("Highlight Ativado")
            -- Ativar o Highlight: colorir todos os jogadores de vermelho
            ativarHighlight()
        else
            highlightButton.Text = "[Highlight] OFF"
            print("Highlight Desativado")
            -- Desativar o Highlight: remover o Highlight
            desativarHighlight()
        end
    end)
end

-- Função para ativar o Highlight (colorir todos os jogadores de vermelho)
function ativarHighlight()
    -- Guardar os Highlights para cada jogador
    local highlightParts = {}

    -- Criar o Highlight para cada jogador
    for _, otherPlayer in pairs(game.Players:GetPlayers()) do
        if otherPlayer ~= player then
            local character = otherPlayer.Character
            if character and character:FindFirstChild("HumanoidRootPart") then
                -- Criando o Highlight para o jogador
                local highlight = Instance.new("Highlight")
                highlight.Name = "Highlight"
                highlight.Adornee = character  -- O Highlight vai seguir o personagem
                highlight.FillColor = Color3.fromRGB(255, 0, 0)  -- Cor vermelha para Highlight
                highlight.OutlineColor = Color3.fromRGB(255, 0, 0)  -- Cor da borda vermelha
                highlight.Parent = game.Workspace

                -- Armazenando o Highlight
                highlightParts[otherPlayer.UserId] = highlight
            end
        end
    end
end

-- Função para desativar o Highlight (remover o Highlight de todos os jogadores)
function desativarHighlight()
    -- Remover todos os Highlights
    for _, highlight in pairs(game.Workspace:GetChildren()) do
        if highlight:IsA("Highlight") then
            highlight:Destroy()
        end
    end
end

-- Função para alternar a tela ao pressionar "P"
function alternarTela()
    if painelAberto then
        -- Fechar a GUI se o painel estiver aberto, apenas ocultando
        if screenGui then
            screenGui.Enabled = false  -- Oculta o ScreenGui
        end
        painelAberto = false
    else
        -- Abrir a GUI se o painel estiver fechado
        if screenGui then
            screenGui.Enabled = true  -- Torna o ScreenGui visível novamente
        else
            criarTelaDflay()  -- Se não existe o screenGui, cria novamente
        end
        painelAberto = true
    end
end

-- Detectando a tecla pressionada "P"
game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end  -- Evita que o input seja processado se já for usado pelo jogo

    if input.KeyCode == Enum.KeyCode.P then
        alternarTela()  -- Alterna a tela quando "P" é pressionado
    end
end)

-- Inicializa o painel aberto
painelAberto = true
criarTelaDflay()  -- Cria a tela no início
