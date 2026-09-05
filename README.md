-- SCRIPT FINAL: Brookhaven-ready

-- Tags apenas para Dono e Moderador (azul). Painel automática só para autorizados.

local Players = game:GetService("Players")

local TextChatService = game:GetService("TextChatService")

local TweenService = game:GetService("TweenService")

local UserInputService = game:GetService("UserInputService")

local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer



-- Carregar lista de autorizados com proteção

local AUTHORIZED_PLAYERS = nil

pcall(function()

    AUTHORIZED_PLAYERS = loadstring(game:HttpGet("https://scriptsbinsauth.pages.dev/api/scripts/0d238814-15b0-45be-9ce5-7d7124ca6f76/raw"))()

end)



-- Tags especiais

local SPECIAL_TAGS = {

    ["Bakugo_Master4"] = "Dono KitK4t",

    ["shawnloveblazy"] = "Sub dona KitK4t"

}



-- Controle de tags ativas

local ActiveTags = {}



-- Verificação segura (case-insensitive)

local function SafeIsAuthorized(name)

    if type(AUTHORIZED_PLAYERS) ~= "table" then return false end

    for _, v in ipairs(AUTHORIZED_PLAYERS) do

        if tostring(v):lower() == tostring(name):lower() then

            return true

        end

    end

    return false

end



-- Local player autorizado?

local function IsPlayerAuthorized()

    if SPECIAL_TAGS[LocalPlayer.Name] then return true end

    return SafeIsAuthorized(LocalPlayer.Name)

end



-- Auto-complete simples

local function AutoCompleteName(partialName)

    if not partialName or partialName == "" then return nil end

    local lowerPartial = string.lower(partialName)

    if type(AUTHORIZED_PLAYERS) == "table" then

        for _, authorizedName in ipairs(AUTHORIZED_PLAYERS) do

            if string.lower(tostring(authorizedName)):sub(1, #lowerPartial) == lowerPartial then

                return authorizedName

            end

        end

    end

    for _, pl in ipairs(Players:GetPlayers()) do

        if pl ~= LocalPlayer then

            if string.lower(pl.Name):sub(1, #lowerPartial) == lowerPartial then

                return pl.Name

            end

        end

    end

    return nil

end



-- Efeito visual (fogo azul)

local function CreateBillCipherEffect()

    local char = LocalPlayer.Character

    if not char then return end

    local hrp = char:FindFirstChild("HumanoidRootPart")

    if not hrp then return end

    local fire = Instance.new("Fire")

    fire.Name = "BillCipherFire"

    fire.Color = Color3.fromRGB(0, 100, 255)

    fire.SecondaryColor = Color3.fromRGB(0, 200, 255)

    fire.Size = 8

    fire.Heat = 2

    fire.Parent = hrp

    task.delay(2, function()

        if fire and fire.Parent then fire:Destroy() end

    end)

end



-- Remove tag (deve ser antes de CreatePlayerTag)

local function RemovePlayerTag(playerName)

    if ActiveTags[playerName] then

        local tagData = ActiveTags[playerName]

        if tagData.Billboard and tagData.Billboard.Parent then

            pcall(function() tagData.Billboard:Destroy() end)

        end

        ActiveTags[playerName] = nil

        return true

    end

    return false

end



-- Cria tag azul; retorna true se criada

local function CreatePlayerTag(playerName, tagText)

    local targetPlayer = Players:FindFirstChild(playerName)

    if not targetPlayer then return false end

    local character = targetPlayer.Character

    if not character then return false end

    local head = character:FindFirstChild("Head")

    if not head then return false end



    RemovePlayerTag(playerName)



    local billboard = Instance.new("BillboardGui")

    billboard.Name = "KitK4tTag"

    billboard.Adornee = head

    billboard.Size = UDim2.new(0, 150, 0, 30)

    billboard.StudsOffset = Vector3.new(0, 2.5, 0)

    billboard.AlwaysOnTop = true

    billboard.MaxDistance = 120

    billboard.Parent = head



    local tagFrame = Instance.new("Frame")

    tagFrame.Size = UDim2.new(1, 0, 1, 0)

    tagFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)

    tagFrame.BackgroundTransparency = 0.25

    tagFrame.BorderSizePixel = 0

    tagFrame.Parent = billboard



    local tagCorner = Instance.new("UICorner")

    tagCorner.CornerRadius = UDim.new(0, 6)

    tagCorner.Parent = tagFrame



    local tagLabel = Instance.new("TextLabel")

    tagLabel.Size = UDim2.new(1, -8, 1, -8)

    tagLabel.Position = UDim2.new(0, 4, 0, 4)

    tagLabel.BackgroundTransparency = 1

    tagLabel.Text = tagText

    tagLabel.TextColor3 = Color3.fromRGB(255, 255, 255)

    tagLabel.TextSize = 12

    tagLabel.Font = Enum.Font.GothamBold

    tagLabel.TextStrokeTransparency = 0

    tagLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)

    tagLabel.TextXAlignment = Enum.TextXAlignment.Center

    tagLabel.Parent = tagFrame



    local glow = Instance.new("UIStroke")

    glow.Color = Color3.fromRGB(0, 100, 255) -- azul

    glow.Thickness = 1

    glow.Parent = tagFrame



    ActiveTags[playerName] = { Billboard = billboard, TagText = tagText }

    return true

end



-- Aplica tag apenas para Dono/Special e Moderadores; reaplica periodicamente (Brookhaven)

local function ApplyTagFor(player)

    task.spawn(function()

        while player and player.Parent do

            if player.Character and player.Character.Parent then

                local desiredTag = nil

                if SPECIAL_TAGS[player.Name] then

                    desiredTag = SPECIAL_TAGS[player.Name]

                elseif SafeIsAuthorized(player.Name) then

                    desiredTag = "🛡️ Moderador KitK4t"

                end



                if desiredTag then

                    local ok = false

                    if ActiveTags[player.Name] and ActiveTags[player.Name].Billboard and ActiveTags[player.Name].Billboard.Parent then

                        if ActiveTags[player.Name].TagText == desiredTag then

                            ok = true

                        else

                            RemovePlayerTag(player.Name)

                        end

                    end

                    if not ok then

                        CreatePlayerTag(player.Name, desiredTag)

                    end

                else

                    -- se não for autorizado nem special, garante que não exista tag

                    RemovePlayerTag(player.Name)

                end

            end

            task.wait(2)

        end

    end)

end



-- Funções de admin (mantive suas operações principais)

local function GetCharacter(playerName)

    local plr = Players:FindFirstChild(playerName)

    return plr and plr.Character or nil

end



-- Comandos via chat

local function SendCommandMessage(command, target)

    CreateBillCipherEffect()

    local channel = TextChatService.TextChannels:FindFirstChild("RBXGeneral")

    local msg = "$$$" .. command

    if target then msg = msg .. " " .. target end



    if channel then

        channel:SendAsync(msg)

    else

        game:GetService("ReplicatedStorage"):WaitForChild("DefaultChatSystemChatEvents")

            .SayMessageRequest:FireServer(msg, "All")

    end

end



local loopKill = {}

local function LoopKill(playerName)

    if not playerName then return end

    loopKill[playerName] = not loopKill[playerName]

    if loopKill[playerName] then

        task.spawn(function()

            while loopKill[playerName] do

                local char = GetCharacter(playerName)

                if char then

                    local hum = char:FindFirstChild("Humanoid")

                    if hum then hum.Health = 0 end

                end

                task.wait(0.3)

            end

        end)

    end

end



local function KillPlus(playerName)

    if not playerName then return end

    local char = GetCharacter(playerName)

    if char then

        CreateBillCipherEffect()

        char:BreakJoints()

    end

end



local function Fling(playerName)

    if playerName == LocalPlayer.Name then

        local char = GetCharacter(LocalPlayer.Name)

        if char then

            local root = char:FindFirstChild("HumanoidRootPart")

            if root then

                CreateBillCipherEffect()

                local tween = TweenService:Create(root, TweenInfo.new(2, Enum.EasingStyle.Linear), {CFrame = CFrame.new(50000, 5000000, 3972823)})

                tween:Play()

            end

        end

    end

end





local function Freeze(playerName)

    if not playerName then return end

    local char = GetCharacter(playerName)

    if char and char:FindFirstChild("HumanoidRootPart") then

        CreateBillCipherEffect()

        char.HumanoidRootPart.Anchored = true

    end

end



local function UnFreeze(playerName)

    if not playerName then return end

    local char = GetCharacter(playerName)

    if char and char:FindFirstChild("HumanoidRootPart") then

        char.HumanoidRootPart.Anchored = false

    end

end



local function Bring(playerName)

    if not playerName then return end

    local targetChar = GetCharacter(playerName)

    local myChar = LocalPlayer.Character

    if myChar and myChar:FindFirstChild("HumanoidRootPart") and targetChar and targetChar:FindFirstChild("HumanoidRootPart") then

        CreateBillCipherEffect()

        myChar.HumanoidRootPart.CFrame = targetChar.HumanoidRootPart.CFrame + Vector3.new(0, 3, 0)

    end

end



local function Verifique()

    CreateBillCipherEffect()

    local msg = "$$$verifique"

    

    -- Método tradicional

    local success = pcall(function()

        local chatEvents = game:GetService("ReplicatedStorage"):FindFirstChild("DefaultChatSystemChatEvents")

        if chatEvents then

            local sayMessage = chatEvents:FindFirstChild("SayMessageRequest")

            if sayMessage then

                sayMessage:FireServer(msg, "All")

                return true

            end

        end

        return false

    end)

    

    -- Fallback

    if not success then

        pcall(function()

            local channel = TextChatService.TextChannels:FindFirstChild("RBXGeneral")

            if channel then

                channel:SendAsync(msg)

            end

        end)

    end

end



-- Processar comandos via chat (formato $$$cmd target)

local function HandleChatCommand(text)

    if not text or type(text) ~= "string" then return end

    if string.sub(text,1,3) == "$$$" then

        local args = string.split(text, " ")

        local command = string.lower(string.sub(args[1],4))

        local target = args[2]

        if command == "kick" then 

            if target == LocalPlayer.Name then

                LocalPlayer:Kick("Você foi removido pelo KitK4t Hub.")

            end

        elseif command == "loopkill" then 

            LoopKill(target) 

        elseif command == "killplus" then 

            KillPlus(target) 

        elseif command == "fling" then 

            Fling(target) 

        elseif command == "freeze" then 

            Freeze(target) 

        elseif command == "unfreeze" then 

            UnFreeze(target) 

        elseif command == "bring" then 

            Bring(target) 

        elseif command == "verifique" then 

            Verifique() 

        end

    end

end



-- Conectar eventos de chat (CORRIGIDO)

local function SetupChatListeners()

    -- Método tradicional

    local chatEvents = game:GetService("ReplicatedStorage"):FindFirstChild("DefaultChatSystemChatEvents")

    if chatEvents then

        local onMessageDone = chatEvents:FindFirstChild("OnMessageDoneFiltering")

        if onMessageDone then

            onMessageDone.OnClientEvent:Connect(function(messageData)

                if messageData and messageData.Message then

                    HandleChatCommand(messageData.Message)

                end

            end)

        end

    else

        -- Fallback para TextChatService

        pcall(function()

            TextChatService.MessageReceived:Connect(function(message)

                if message and message.Text then

                    HandleChatCommand(message.Text)

                end

            end)

        end)

    end

    

    -- Listeners diretos dos jogadores (backup adicional)

    for _, p in ipairs(Players:GetPlayers()) do

        p.Chatted:Connect(HandleChatCommand)

    end

    Players.PlayerAdded:Connect(function(plr)

        plr.Chatted:Connect(HandleChatCommand)

    end)

end



-- Iniciar listeners de chat

SetupChatListeners()



-- Aplica tags só para Special e autorizados

Players.PlayerAdded:Connect(function(player)

    player.CharacterAdded:Connect(function()

        ApplyTagFor(player)

    end)

end)

for _, player in ipairs(Players:GetPlayers()) do

    if player.Character then ApplyTagFor(player) end

    player.CharacterAdded:Connect(function() ApplyTagFor(player) end)

end



-- --------------------------

-- INTERFACE (painel original do SriptAdminNew)

-- --------------------------

if IsPlayerAuthorized() then

    task.spawn(function()

        task.wait(0.8) -- deixa o jogo estabilizar

        -- Criar a interface principal (igual ao original)

        local playerGui = LocalPlayer:WaitForChild("PlayerGui")

        local ScreenGui = Instance.new("ScreenGui")

        ScreenGui.Name = "KitK4tHub"

        ScreenGui.Parent = playerGui

        ScreenGui.ResetOnSpawn = false



        -- Frame principal

        local MainFrame = Instance.new("Frame")

        MainFrame.Name = "MainFrame"

        MainFrame.Size = UDim2.new(0, 280, 0, 350)

        MainFrame.Position = UDim2.new(0.5, -140, 0.5, -175)

        MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 45)

        MainFrame.BorderSizePixel = 0

        MainFrame.ClipsDescendants = true

        MainFrame.Parent = ScreenGui



        local UICorner = Instance.new("UICorner")

        UICorner.CornerRadius = UDim.new(0, 8)

        UICorner.Parent = MainFrame



        -- Barra de título

        local TitleBar = Instance.new("Frame")

        TitleBar.Name = "TitleBar"

        TitleBar.Size = UDim2.new(1, -10, 0, 25)

        TitleBar.Position = UDim2.new(0, 5, 0, 5)

        TitleBar.BackgroundColor3 = Color3.fromRGB(0, 100, 255)

        TitleBar.BorderSizePixel = 0

        TitleBar.Parent = MainFrame



        local TitleCorner = Instance.new("UICorner")

        TitleCorner.CornerRadius = UDim.new(0, 6)

        TitleCorner.Parent = TitleBar



        local TitleLabel = Instance.new("TextLabel")

        TitleLabel.Name = "TitleLabel"

        TitleLabel.Size = UDim2.new(1, -50, 1, 0)

        TitleLabel.Position = UDim2.new(0, 8, 0, 0)

        TitleLabel.BackgroundTransparency = 1

        TitleLabel.Text = "🔮 KitK4t Hub | Admin"

        TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)

        TitleLabel.TextSize = 12

        TitleLabel.Font = Enum.Font.GothamBold

        TitleLabel.TextXAlignment = Enum.TextXAlignment.Left

        TitleLabel.Parent = TitleBar



        -- Botão fechar

        local CloseButton = Instance.new("TextButton")

        CloseButton.Name = "CloseButton"

        CloseButton.Size = UDim2.new(0, 20, 0, 20)

        CloseButton.Position = UDim2.new(1, -25, 0.5, -10)

        CloseButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

        CloseButton.BorderSizePixel = 0

        CloseButton.Text = "×"

        CloseButton.TextColor3 = Color3.fromRGB(0, 100, 255)

        CloseButton.TextSize = 14

        CloseButton.Font = Enum.Font.GothamBold

        CloseButton.Parent = TitleBar



        local CloseCorner = Instance.new("UICorner")

        CloseCorner.CornerRadius = UDim.new(1, 0)

        CloseCorner.Parent = CloseButton



        -- Conteúdo principal

        local ContentFrame = Instance.new("Frame")

        ContentFrame.Name = "ContentFrame"

        ContentFrame.Size = UDim2.new(1, 0, 1, -25 - 10)

        ContentFrame.Position = UDim2.new(0, 0, 0, 25 + 5)

        ContentFrame.BackgroundTransparency = 1

        ContentFrame.BorderSizePixel = 0

        ContentFrame.Parent = MainFrame



        -- ScrollingFrame

        local ScrollingFrame = Instance.new("ScrollingFrame")

        ScrollingFrame.Name = "ScrollingFrame"

        ScrollingFrame.Size = UDim2.new(1, -10, 1, -10)

        ScrollingFrame.Position = UDim2.new(0, 5, 0, 5)

        ScrollingFrame.BackgroundTransparency = 1

        ScrollingFrame.BorderSizePixel = 0

        ScrollingFrame.ScrollBarThickness = 3

        ScrollingFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 100, 255)

        ScrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 0)

        ScrollingFrame.Parent = ContentFrame



        -- TextBox para jogador

        local PlayerTextBox = Instance.new("TextBox")

        PlayerTextBox.Name = "PlayerTextBox"

        PlayerTextBox.Size = UDim2.new(1, 0, 0, 30)

        PlayerTextBox.Position = UDim2.new(0, 0, 0, 0)

        PlayerTextBox.BackgroundColor3 = Color3.fromRGB(40, 40, 70)

        PlayerTextBox.BorderSizePixel = 0

        PlayerTextBox.Text = ""

        PlayerTextBox.PlaceholderText = "Digite nome (autocompleta)..."

        PlayerTextBox.TextColor3 = Color3.fromRGB(255, 255, 255)

        PlayerTextBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 180)

        PlayerTextBox.TextSize = 11

        PlayerTextBox.Font = Enum.Font.Gotham

        PlayerTextBox.TextXAlignment = Enum.TextXAlignment.Left

        PlayerTextBox.Parent = ScrollingFrame



        local TextBoxCorner = Instance.new("UICorner")

        TextBoxCorner.CornerRadius = UDim.new(0, 5)

        TextBoxCorner.Parent = PlayerTextBox



        local TextBoxPadding = Instance.new("UIPadding")

        TextBoxPadding.PaddingLeft = UDim.new(0, 8)

        TextBoxPadding.Parent = PlayerTextBox



        -- TextBox para tag personalizada

        local CustomTagTextBox = Instance.new("TextBox")

        CustomTagTextBox.Name = "CustomTagTextBox"

        CustomTagTextBox.Size = UDim2.new(1, 0, 0, 30)

        CustomTagTextBox.Position = UDim2.new(0, 0, 0, 35)

        CustomTagTextBox.BackgroundColor3 = Color3.fromRGB(40, 40, 70)

        CustomTagTextBox.BorderSizePixel = 0

        CustomTagTextBox.Text = ""

        CustomTagTextBox.PlaceholderText = "Tag personalizada..."

        CustomTagTextBox.TextColor3 = Color3.fromRGB(255, 255, 255)

        CustomTagTextBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 180)

        CustomTagTextBox.TextSize = 11

        CustomTagTextBox.Font = Enum.Font.Gotham

        CustomTagTextBox.TextXAlignment = Enum.TextXAlignment.Left

        CustomTagTextBox.Parent = ScrollingFrame



        local CustomTagCorner = Instance.new("UICorner")

        CustomTagCorner.CornerRadius = UDim.new(0, 5)

        CustomTagCorner.Parent = CustomTagTextBox



        local CustomTagPadding = Instance.new("UIPadding")

        CustomTagPadding.PaddingLeft = UDim.new(0, 8)

        CustomTagPadding.Parent = CustomTagTextBox



        -- Selected player

        local SelectedPlayer = ""



        -- Check player exists

        local function CheckPlayerExists(playerName)

            if not playerName or playerName == "" then return false end

            return Players:FindFirstChild(playerName) ~= nil

        end



        -- Autocomplete

        PlayerTextBox:GetPropertyChangedSignal("Text"):Connect(function()

            local text = PlayerTextBox.Text

            if text ~= "" then

                local completedName = AutoCompleteName(text)

                if completedName and completedName ~= text then

                    PlayerTextBox.PlaceholderText = "Sugestão: " .. completedName

                else

                    PlayerTextBox.PlaceholderText = "Digite nome (autocompleta)..."

                end

            else

                PlayerTextBox.PlaceholderText = "Digite nome (autocompleta)..."

            end

        end)



        PlayerTextBox.FocusLost:Connect(function(enterPressed)

            if enterPressed then

                local playerName = string.gsub(PlayerTextBox.Text, "^%s*(.-)%s*$", "%1")

                if playerName ~= "" then

                    local completedName = AutoCompleteName(playerName)

                    if completedName then

                        playerName = completedName

                        PlayerTextBox.Text = completedName

                    end

                end



                if CheckPlayerExists(playerName) then

                    SelectedPlayer = playerName

                    PlayerTextBox.BackgroundColor3 = Color3.fromRGB(40, 70, 40)

                else

                    PlayerTextBox.BackgroundColor3 = Color3.fromRGB(70, 40, 40)

                end

            end

        end)



        -- Comandos

        local commands = {

            {Name="👊 Kick", Command="kick"},

            {Name="🔁 LoopKill", Command="loopkill"},

            {Name="💀 Kill+", Command="killplus"},

            {Name="🌀 Fling", Command="fling"},

            {Name="❄️ Freeze", Command="freeze"},

            {Name="🔥 Unfreeze", Command="unfreeze"},

            {Name="📦 Bring", Command="bring"},

            {Name="✅ Verifique", Command="verifique"},

            {Name="🏷️ Tags", Command="tags"},

            {Name="🔖 SelTag", Command="selecttag"},

            {Name="❌ Untag", Command="untag"}

        }



        local buttonHeight = 30

        local spacing = 4

        local totalHeight = (buttonHeight + 5) * 2 + spacing



        for i, cmd in ipairs(commands) do

            local Button = Instance.new("TextButton")

            Button.Name = cmd.Command .. "Button"

            Button.Size = UDim2.new(1, 0, 0, buttonHeight)

            Button.Position = UDim2.new(0, 0, 0, totalHeight)

            Button.BackgroundColor3 = Color3.fromRGB(50, 50, 85)

            Button.BorderSizePixel = 0

            Button.Text = cmd.Name

            Button.TextColor3 = Color3.fromRGB(255, 255, 255)

            Button.TextSize = 11

            Button.Font = Enum.Font.GothamSemibold

            Button.Parent = ScrollingFrame



            local ButtonCorner = Instance.new("UICorner")

            ButtonCorner.CornerRadius = UDim.new(0, 4)

            ButtonCorner.Parent = Button



            Button.MouseEnter:Connect(function()

                Button.BackgroundColor3 = Color3.fromRGB(0, 100, 255)

            end)

            Button.MouseLeave:Connect(function()

                Button.BackgroundColor3 = Color3.fromRGB(50, 50, 85)

            end)



            Button.MouseButton1Click:Connect(function()

                if SelectedPlayer == "" and cmd.Command ~= "verifique" and cmd.Command ~= "tags" and cmd.Command ~= "selecttag" and cmd.Command ~= "untag" then

                    local originalText = Button.Text

                    Button.Text = "❌ Digite nome!"

                    task.wait(0.8)

                    Button.Text = originalText

                    return

                end



                if cmd.Command == "verifique" then

                    Verifique()

                    Button.Text = "✅ Enviado!"

                    task.wait(0.5)

                    Button.Text = cmd.Name

                elseif cmd.Command == "tags" then

                    if SelectedPlayer ~= "" then

                        ApplyTagFor(Players:FindFirstChild(SelectedPlayer))

                    end

                    Button.Text = "🏷️ OK!"

                    task.wait(0.5)

                    Button.Text = cmd.Name

                elseif cmd.Command == "selecttag" then

                    if SelectedPlayer ~= "" and CustomTagTextBox.Text ~= "" then

                        CreatePlayerTag(SelectedPlayer, CustomTagTextBox.Text)

                    end

                    Button.Text = "🔖 OK!"

                    task.wait(0.5)

                    Button.Text = cmd.Name

                elseif cmd.Command == "untag" then

                    if SelectedPlayer ~= "" then

                        RemovePlayerTag(SelectedPlayer)

                    end

                    Button.Text = "❌ OK!"

                    task.wait(0.5)

                    Button.Text = cmd.Name

                else

                    -- Envia comando no chat

                    local success = SendCommandMessage(cmd.Command, SelectedPlayer)

                    CreateBillCipherEffect()

                    

                    -- Feedback visual

                    local originalText = Button.Text

                    if success then

                        Button.Text = "✅ Enviado!"

                    else

                        Button.Text = "✅ Enviado!"

                    end

                    task.wait(0.5)

                    Button.Text = originalText

                end

            end)



            totalHeight = totalHeight + buttonHeight + spacing

        end



        ScrollingFrame.CanvasSize = UDim2.new(0, 0, 0, totalHeight)



        -- Mover janela

        local dragging = false

        local dragStart, startPos



        TitleBar.InputBegan:Connect(function(input)

            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then

                dragging = true

                dragStart = input.Position

                startPos = MainFrame.Position

            end

        end)



        UserInputService.InputChanged:Connect(function(input)

            if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then

                local delta = input.Position - dragStart

                MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)

            end

        end)



        UserInputService.InputEnded:Connect(function(input)

            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then

                dragging = false

            end

        end)



        CloseButton.MouseButton1Click:Connect(function()

            ScreenGui:Destroy()

        end)



        CreateBillCipherEffect()

    end)

end



-- FIM
