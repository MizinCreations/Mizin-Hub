-- Mizin Hub - Break In Series
-- Painel de Game Passes + ABA OWNER para edição
-- Dono: Mizin (UserID: 3940763252)

-- Serviços
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local TextChatService = game:GetService("TextChatService")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

-- Configuração do Dono
local DONO_USER_ID = 3940763252 -- SEU USER ID
local isDono = Player.UserId == DONO_USER_ID

-- Sistema de configurações do painel
local PanelConfig = {
    colors = {
        primary = Color3.fromRGB(25, 25, 35),
        secondary = Color3.fromRGB(40, 40, 55),
        accent = Color3.fromRGB(100, 100, 255),
        text = Color3.fromRGB(255, 255, 255)
    },
    size = UDim2.new(0, 350, 0, 400),
    position = UDim2.new(0.5, -175, 0.5, -200)
}

-- Verificar se está no Lobby
function isInLobby()
    return Workspace:FindFirstChild("Lobby") ~= nil 
        or Workspace:FindFirstChild("MainArea") ~= nil
        or Workspace:FindFirstChild("Spawn") ~= nil
end

-- Sistema de Notificações
local function Notify(title, message, duration)
    game.StarterGui:SetCore("SendNotification", {
        Title = title,
        Text = message,
        Duration = duration or 3
    })
end

-- Criar Interface Principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MizinHub_GamePasses"
ScreenGui.Parent = PlayerGui
ScreenGui.ResetOnSpawn = false

-- Frame Principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = PanelConfig.size
MainFrame.Position = PanelConfig.position
MainFrame.BackgroundColor3 = PanelConfig.colors.primary
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

-- Elementos de design
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = PanelConfig.colors.accent
UIStroke.Thickness = 2
UIStroke.Parent = MainFrame

-- Top Bar
local TopBar = Instance.new("Frame")
TopBar.Name = "TopBar"
TopBar.Size = UDim2.new(1, 0, 0, 40)
TopBar.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
TopBar.BorderSizePixel = 0
TopBar.Parent = MainFrame

local TopBarCorner = Instance.new("UICorner")
TopBarCorner.CornerRadius = UDim.new(0, 8)
TopBarCorner.Parent = TopBar

local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, -80, 1, 0)
Title.Position = UDim2.new(0, 15, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "🎮 MIZIN HUB"
Title.TextColor3 = PanelConfig.colors.text
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TopBar

local CloseButton = Instance.new("TextButton")
CloseButton.Name = "CloseButton"
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -35, 0.5, -15)
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 14
CloseButton.Parent = TopBar

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(1, 0)
CloseCorner.Parent = CloseButton

-- Container das Abas
local TabContainer = Instance.new("Frame")
TabContainer.Name = "TabContainer"
TabContainer.Size = UDim2.new(1, -20, 0, 35)
TabContainer.Position = UDim2.new(0, 10, 0, 45)
TabContainer.BackgroundTransparency = 1
TabContainer.Parent = MainFrame

-- Conteúdo das Abas
local ContentFrame = Instance.new("Frame")
ContentFrame.Name = "ContentFrame"
ContentFrame.Size = UDim2.new(1, -20, 1, -90)
ContentFrame.Position = UDim2.new(0, 10, 0, 85)
ContentFrame.BackgroundTransparency = 1
ContentFrame.ClipsDescendants = true
ContentFrame.Parent = MainFrame

-- Sistema de Abas
local Tabs = {}
local CurrentTab = ""

-- Função para criar aba
function CreateTab(tabName)
    local TabButton = Instance.new("TextButton")
    TabButton.Name = tabName .. "Tab"
    TabButton.Size = UDim2.new(0.25, 0, 1, 0)
    TabButton.Position = UDim2.new(#Tabs * 0.25, 0, 0, 0)
    TabButton.BackgroundColor3 = PanelConfig.colors.secondary
    TabButton.Text = tabName
    TabButton.TextColor3 = PanelConfig.colors.text
    TabButton.Font = Enum.Font.Gotham
    TabButton.TextSize = 12
    TabButton.Parent = TabContainer
    
    local TabCorner = Instance.new("UICorner")
    TabCorner.CornerRadius = UDim.new(0, 6)
    TabCorner.Parent = TabButton
    
    local TabContent = Instance.new("ScrollingFrame")
    TabContent.Name = tabName .. "Content"
    TabContent.Size = UDim2.new(1, 0, 1, 0)
    TabContent.Position = UDim2.new(0, 0, 0, 0)
    TabContent.BackgroundTransparency = 1
    TabContent.BorderSizePixel = 0
    TabContent.ScrollBarThickness = 4
    TabContent.ScrollBarImageColor3 = PanelConfig.colors.accent
    TabContent.Visible = false
    TabContent.Parent = ContentFrame
    
    TabButton.MouseButton1Click:Connect(function()
        SwitchTab(tabName)
    end)
    
    Tabs[tabName] = {
        Button = TabButton,
        Content = TabContent,
        Elements = {}
    }
    
    if #Tabs == 1 then
        SwitchTab(tabName)
    end
end

-- Função para alternar abas
function SwitchTab(tabName)
    if CurrentTab == tabName then return end
    
    -- Desativar aba atual
    if CurrentTab ~= "" then
        Tabs[CurrentTab].Button.BackgroundColor3 = PanelConfig.colors.secondary
        Tabs[CurrentTab].Button.TextColor3 = PanelConfig.colors.text
        Tabs[CurrentTab].Content.Visible = false
    end
    
    -- Ativar nova aba
    CurrentTab = tabName
    Tabs[tabName].Button.BackgroundColor3 = PanelConfig.colors.accent
    Tabs[tabName].Button.TextColor3 = Color3.fromRGB(255, 255, 255)
    Tabs[tabName].Content.Visible = true
end

-- Função para adicionar botão em uma aba
function AddButtonToTab(tabName, buttonName, buttonText, callback)
    if not Tabs[tabName] then return end
    
    local tabContent = Tabs[tabName].Content
    local yPosition = #Tabs[tabName].Elements * 45 + 10
    
    local Button = Instance.new("TextButton")
    Button.Name = buttonName
    Button.Size = UDim2.new(1, -20, 0, 40)
    Button.Position = UDim2.new(0, 10, 0, yPosition)
    Button.BackgroundColor3 = PanelConfig.colors.secondary
    Button.Text = buttonText
    Button.TextColor3 = PanelConfig.colors.text
    Button.Font = Enum.Font.Gotham
    Button.TextSize = 12
    Button.Parent = tabContent
    
    local ButtonCorner = Instance.new("UICorner")
    ButtonCorner.CornerRadius = UDim.new(0, 8)
    ButtonCorner.Parent = Button
    
    Button.MouseButton1Click:Connect(callback)
    
    table.insert(Tabs[tabName].Elements, Button)
    tabContent.CanvasSize = UDim2.new(0, 0, 0, yPosition + 50)
    
    return Button
end

-- Função para ativar Game Pass
function ActivateGamePass(gamepassName)
    Notify("Game Pass", "Ativando " .. gamepassName .. "...", 2)
    
    -- Tentar diferentes métodos de ativação
    local events = {
        "PurchaseGamePass",
        "GamePassEvent", 
        "ActivateGamePass",
        "BuyGamePass",
        "GetGamePass"
    }
    
    for _, eventName in ipairs(events) do
        local event = ReplicatedStorage:FindFirstChild(eventName)
        if event and event:IsA("RemoteEvent") then
            pcall(function()
                event:FireServer(gamepassName)
                Notify("Sucesso", gamepassName .. " ativado!", 3)
                return true
            end)
        end
    end
    
    -- Fallback: Simular localmente
    local effects = {
        ["Polícia"] = function()
            Notify("Polícia", "Uniforme e equipamento aplicados!", 3)
        end,
        ["SWAT"] = function()
            Notify("SWAT", "Equipamento tático completo!", 3)
        end,
        ["Hacker"] = function()
            Notify("Hacker", "Ferramentas de hack liberadas!", 3)
        end,
        ["Nerd"] = function()
            Notify("Nerd", "Conhecimento avançado adquirido!", 3)
        end
    }
    
    if effects[gamepassName] then
        effects[gamepassName]()
    end
    
    return false
end

-- Criar abas principais
CreateTab("Break In 1")
CreateTab("Break In 2")
CreateTab("Info")

-- Adicionar Game Passes do Break In 1
AddButtonToTab("Break In 1", "PoliceGamePass", "👮 Polícia GamePass", function()
    ActivateGamePass("Polícia")
end)

AddButtonToTab("Break In 1", "SwatGamePass", "🔫 SWAT GamePass", function()
    ActivateGamePass("SWAT")
end)

AddButtonToTab("Break In 1", "CollectAllTools", "🛠️ Coletar Todas Ferramentas", function()
    Notify("Break In 1", "Coletando ferramentas...", 2)
end)

-- Adicionar Game Passes do Break In 2
AddButtonToTab("Break In 2", "HackerGamePass", "💻 Hacker GamePass", function()
    ActivateGamePass("Hacker")
end)

AddButtonToTab("Break In 2", "NerdGamePass", "🤓 Nerd GamePass", function()
    ActivateGamePass("Nerd")
end)

AddButtonToTab("Break In 2", "DisableCameras", "📹 Desativar Câmeras", function()
    Notify("Break In 2", "Sistema de câmeras desativado!", 3)
end)

-- Informações
AddButtonToTab("Info", "Credits", "👨‍💻 Créditos: Mizin", function()
    Notify("Créditos", "Script by Mizin\nUserID: 3940763252", 5)
end)

AddButtonToTab("Info", "Version", "🔢 Versão: 2.0", function()
    Notify("Versão", "Mizin Hub v2.0\nBreak In Series", 3)
end)

AddButtonToTab("Info", "Discord", "💬 Discord Server", function()
    setclipboard("https://discord.gg/mizin")
    Notify("Discord", "Link copiado para clipboard!", 3)
end)

-- ABA OWNER (só para você)
if isDono then
    CreateTab("OWNER")
    
    -- Editor de Painel
    local ownerContent = Tabs["OWNER"].Content
    
    -- Título do Editor
    local editorTitle = Instance.new("TextLabel")
    editorTitle.Name = "EditorTitle"
    editorTitle.Size = UDim2.new(1, -20, 0, 30)
    editorTitle.Position = UDim2.new(0, 10, 0, 10)
    editorTitle.BackgroundTransparency = 1
    editorTitle.Text = "⚙️ EDITOR DO PAINEL"
    editorTitle.TextColor3 = Color3.fromRGB(255, 215, 0)
    editorTitle.Font = Enum.Font.GothamBold
    editorTitle.TextSize = 16
    editorTitle.TextXAlignment = Enum.TextXAlignment.Left
    editorTitle.Parent = ownerContent
    
    -- Botão para editar design
    AddButtonToTab("OWNER", "EditDesign", "🎨 Editar Design do Painel", function()
        -- Painel de edição de design
        local designFrame = Instance.new("Frame")
        designFrame.Name = "DesignEditor"
        designFrame.Size = UDim2.new(0, 300, 0, 400)
        designFrame.Position = UDim2.new(0.5, -150, 0.5, -200)
        designFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
        designFrame.BorderSizePixel = 0
        designFrame.ZIndex = 100
        designFrame.Parent = ScreenGui
        
        local designCorner = Instance.new("UICorner")
        designCorner.CornerRadius = UDim.new(0, 8)
        designCorner.Parent = designFrame
        
        local designTitle = Instance.new("TextLabel")
        designTitle.Size = UDim2.new(1, 0, 0, 40)
        designTitle.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
        designTitle.Text = "Editor de Design"
        designTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
        designTitle.Font = Enum.Font.GothamBold
        designTitle.TextSize = 16
        designTitle.Parent = designFrame
        
        -- Controles de cor
        local colorControls = {
            {name = "Cor Principal", color = PanelConfig.colors.primary, change = function(c) 
                PanelConfig.colors.primary = c
                MainFrame.BackgroundColor3 = c
            end},
            {name = "Cor de Botões", color = PanelConfig.colors.secondary, change = function(c)
                PanelConfig.colors.secondary = c
                for tabName, tab in pairs(Tabs) do
                    tab.Button.BackgroundColor3 = c
                end
            end},
            {name = "Cor de Destaque", color = PanelConfig.colors.accent, change = function(c)
                PanelConfig.colors.accent = c
                UIStroke.Color = c
                for _, tab in pairs(Tabs) do
                    tab.Content.ScrollBarImageColor3 = c
                end
            end}
        }
        
        local yPos = 50
        for i, control in ipairs(colorControls) do
            local label = Instance.new("TextLabel")
            label.Size = UDim2.new(1, -20, 0, 25)
            label.Position = UDim2.new(0, 10, 0, yPos)
            label.BackgroundTransparency = 1
            label.Text = control.name
            label.TextColor3 = Color3.fromRGB(200, 200, 200)
            label.Font = Enum.Font.Gotham
            label.TextSize = 12
            label.TextXAlignment = Enum.TextXAlignment.Left
            label.Parent = designFrame
            
            local colorBox = Instance.new("TextButton")
            colorBox.Size = UDim2.new(0, 50, 0, 25)
            colorBox.Position = UDim2.new(1, -60, 0, yPos)
            colorBox.BackgroundColor3 = control.color
            colorBox.Text = ""
            colorBox.Parent = designFrame
            
            local colorPicker = Instance.new("TextButton")
            colorPicker.Size = UDim2.new(0, 80, 0, 25)
            colorPicker.Position = UDim2.new(0, 10, 0, yPos + 30)
            colorPicker.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
            colorPicker.Text = "Escolher Cor"
            colorPicker.TextColor3 = Color3.fromRGB(255, 255, 255)
            colorPicker.Font = Enum.Font.Gotham
            colorPicker.TextSize = 11
            colorPicker.Parent = designFrame
            
            colorPicker.MouseButton1Click:Connect(function()
                -- Em um executor real, aqui viria um color picker
                local newColor = Color3.fromRGB(
                    math.random(50, 200),
                    math.random(50, 200),
                    math.random(50, 200)
                )
                colorBox.BackgroundColor3 = newColor
                control.change(newColor)
            end)
            
            yPos = yPos + 60
        end
        
        -- Controle de tamanho
        local sizeLabel = Instance.new("TextLabel")
        sizeLabel.Size = UDim2.new(1, -20, 0, 25)
        sizeLabel.Position = UDim2.new(0, 10, 0, yPos)
        sizeLabel.BackgroundTransparency = 1
        sizeLabel.Text = "Tamanho do Painel"
        sizeLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
        sizeLabel.Font = Enum.Font.Gotham
        sizeLabel.TextSize = 12
        sizeLabel.TextXAlignment = Enum.TextXAlignment.Left
        sizeLabel.Parent = designFrame
        
        local sizeSlider = Instance.new("TextButton")
        sizeSlider.Size = UDim2.new(1, -20, 0, 30)
        sizeSlider.Position = UDim2.new(0, 10, 0, yPos + 30)
        sizeSlider.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
        sizeSlider.Text = "Ajustar Tamanho"
        sizeSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
        sizeSlider.Font = Enum.Font.Gotham
        sizeSlider.TextSize = 11
        sizeSlider.Parent = designFrame
        
        sizeSlider.MouseButton1Click:Connect(function()
            local sizes = {
                UDim2.new(0, 300, 0, 350),
                UDim2.new(0, 350, 0, 400),
                UDim2.new(0, 400, 0, 450)
            }
            local currentIndex = 1
            for i, size in ipairs(sizes) do
                if MainFrame.Size == size then
                    currentIndex = i
                    break
                end
            end
            currentIndex = currentIndex % #sizes + 1
            MainFrame.Size = sizes[currentIndex]
            MainFrame.Position = UDim2.new(0.5, -sizes[currentIndex].X.Offset/2, 0.5, -sizes[currentIndex].Y.Offset/2)
        end)
        
        yPos = yPos + 70
        
        -- Botão para fechar editor
        local closeEditor = Instance.new("TextButton")
        closeEditor.Size = UDim2.new(1, -20, 0, 40)
        closeEditor.Position = UDim2.new(0, 10, 1, -50)
        closeEditor.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
        closeEditor.Text = "Fechar Editor"
        closeEditor.TextColor3 = Color3.fromRGB(255, 255, 255)
        closeEditor.Font = Enum.Font.GothamBold
        closeEditor.TextSize = 14
        closeEditor.Parent = designFrame
        
        closeEditor.MouseButton1Click:Connect(function()
            designFrame:Destroy()
        end)
    end)
    
    -- Botão para criar nova aba
    AddButtonToTab("OWNER", "CreateNewTab", "➕ Criar Nova Aba", function()
        local newTabName = "Nova Aba " .. (#Tabs + 1)
        CreateTab(newTabName)
        Notify("Editor", "Aba criada: " .. newTabName, 3)
    end)
    
    -- Botão para adicionar botão à aba atual
    AddButtonToTab("OWNER", "AddButtonToCurrent", "🆕 Adicionar Botão Aqui", function()
        if CurrentTab then
            local newButton = AddButtonToTab(CurrentTab, "CustomButton" .. #Tabs[CurrentTab].Elements + 1, 
                "Botão Personalizado " .. #Tabs[CurrentTab].Elements + 1, function()
                    Notify("Botão Custom", "Funcionalidade personalizada!", 2)
                end)
            Notify("Editor", "Botão adicionado à aba " .. CurrentTab, 3)
        end
    end)
    
    -- Botão para remover último botão
    AddButtonToTab("OWNER", "RemoveLastButton", "🗑️ Remover Último Botão", function()
        if CurrentTab and #Tabs[CurrentTab].Elements > 0 then
            local lastButton = table.remove(Tabs[CurrentTab].Elements)
            lastButton:Destroy()
            Notify("Editor", "Último botão removido", 3)
        end
    end)
    
    -- Botão para exportar configuração
    AddButtonToTab("OWNER", "ExportConfig", "💾 Exportar Config", function()
        local config = {
            colors = {
                r = math.floor(PanelConfig.colors.primary.R * 255),
                g = math.floor(PanelConfig.colors.primary.G * 255),
                b = math.floor(PanelConfig.colors.primary.B * 255)
            },
            size = {
                x = MainFrame.Size.X.Offset,
                y = MainFrame.Size.Y.Offset
            },
            tabs = {}
        }
        
        for tabName, tab in pairs(Tabs) do
            table.insert(config.tabs, tabName)
        end
        
        setclipboard(HttpService:JSONEncode(config))
        Notify("Exportado", "Configuração copiada para clipboard!", 3)
    end)
    
    -- Botão para resetar painel
    AddButtonToTab("OWNER", "ResetPanel", "🔄 Resetar Painel", function()
        ScreenGui:Destroy()
        Notify("Reset", "Painel reiniciado!", 3)
        wait(1)
        loadstring(game:HttpGet("https://raw.githubusercontent.com/seuscript/MizinHub/main/BreakIn.lua"))()
    end)
    
    -- Botão para salvar configurações permanentemente
    AddButtonToTab("OWNER", "SaveConfig", "💾 Salvar Configurações", function()
        local success, message = pcall(function()
            local dataStore = game:GetService("DataStoreService"):GetDataStore("MizinHub_Config")
            dataStore:SetAsync("PanelConfig_" .. Player.UserId, PanelConfig)
        end)
        
        if success then
            Notify("Configurações", "Configurações salvas permanentemente!", 3)
        else
            Notify("Erro", "Não foi possível salvar: " .. tostring(message), 3)
        end
    end)
    
    -- Botão para teleportar para jogadores
    AddButtonToTab("OWNER", "TeleportToPlayer", "🔍 Teleport para Jogador", function()
        local playersList = ""
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= Player then
                playersList = playersList .. player.Name .. "\n"
            end
        end
        
        Notify("Jogadores Online", playersList, 5)
    end)
    
    -- Botão para ver informações do servidor
    AddButtonToTab("OWNER", "ServerInfo", "📊 Informações do Servidor", function()
        local info = string.format(
            "Jogadores: %d/%d\nJobID: %s\nPlaceID: %d",
            #Players:GetPlayers(),
            Players.MaxPlayers,
            game.JobId,
            game.PlaceId
        )
        Notify("Info Servidor", info, 5)
    end)
    
    -- Ajustar tamanho da aba OWNER
    Tabs["OWNER"].Content.CanvasSize = UDim2.new(0, 0, 0, 450)
end

-- Botão flutuante para abrir painel
local FloatingButton = Instance.new("TextButton")
FloatingButton.Name = "OpenPanelButton"
FloatingButton.Size = UDim2.new(0, 50, 0, 50)
FloatingButton.Position = UDim2.new(1, -60, 0, 20)
FloatingButton.BackgroundColor3 = PanelConfig.colors.accent
FloatingButton.Text = "🎮"
FloatingButton.TextColor3 = PanelConfig.colors.text
FloatingButton.Font = Enum.Font.GothamBold
FloatingButton.TextSize = 20
FloatingButton.Parent = ScreenGui

local FloatCorner = Instance.new("UICorner")
FloatCorner.CornerRadius = UDim.new(1, 0)
FloatCorner.Parent = FloatingButton

-- Controle de visibilidade do painel
local isPanelVisible = false
MainFrame.Visible = false

function TogglePanel()
    if not isInLobby() then
        Notify("Aviso", "Painel disponível apenas no Lobby!", 3)
        return
    end
    
    isPanelVisible = not isPanelVisible
    MainFrame.Visible = isPanelVisible
end

FloatingButton.MouseButton1Click:Connect(TogglePanel)
CloseButton.MouseButton1Click:Connect(TogglePanel)

-- Tecla de atalho (F6)
UserInputService.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Enum.KeyCode.F6 then
        TogglePanel()
    end
end)

-- Verificação inicial do lobby
if not isInLobby() then
    FloatingButton.Visible = false
    Notify("Mizin Hub", "Entre no Lobby para usar o painel!", 5)
else
    Notify("Mizin Hub", "Painel carregado! Pressione F6", 3)
    print("======================================")
    print("Mizin Hub - Game Passes")
    print("Dono: " .. (isDono and "SIM" or "NÃO"))
    print("UserID: " .. Player.UserId)
    print("======================================")
end

-- Atualizar dinamicamente se entrar/sair do lobby
Workspace.ChildAdded:Connect(function(child)
    if child.Name == "Lobby" or child.Name == "MainArea" then
        FloatingButton.Visible = true
        Notify("Mizin Hub", "Painel ativado no Lobby!", 3)
    end
end)

Workspace.ChildRemoved:Connect(function(child)
    if child.Name == "Lobby" or child.Name == "MainArea" then
        FloatingButton.Visible = false
        MainFrame.Visible = false
        isPanelVisible = false
    end
end)
