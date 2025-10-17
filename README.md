        active = false,
        highlights = {}
    }
}

-- FunÃ§Ã£o de atualizaÃ§Ã£o de personagem melhorada
local function updateCharacter()
    State.character = State.player.Character or State.player.CharacterAdded:Wait()
    
    if State.character then
        State.humanoid = State.character:WaitForChild("Humanoid", 5)
        State.rootPart = State.character:WaitForChild("HumanoidRootPart", 5)
        
        -- Aplicar configuraÃ§Ãµes salvas
        if State.humanoid then
            State.humanoid.WalkSpeed = State.settings.walkSpeed
        end
    end
    
    return State.character ~= nil and State.humanoid ~= nil and State.rootPart ~= nil
end

-- Setup de personagem
State.player.CharacterAdded:Connect(function()
    wait(1)
    updateCharacter()
    
    -- Reativar funcionalidades se necessÃ¡rio
    if State.movement.noclipActive then
        activateNoclip(true)
    end
end)

-- InicializaÃ§Ã£o
updateCharacter()

-- Criar janela principal
local Window = Library:MakeWindow({
    Title = "High Menu | Mini City RP",
    SubTitle = "by: Keef (Corrigido)",
    LoadText = "Carregando sistema...",
    Flags = "HighMenu_Fixed"
})

Window:AddMinimizeButton({
    Button = { Image = "rbxassetid://92698440248050", BackgroundTransparency = 0 },
    Corner = { CornerRadius = UDim.new(0, 35) },
})

-- =================
-- ABA INFORMAÃ‡Ã•ES
-- =================
local InfoTab = Window:MakeTab({ Title = "Info", Icon = "rbxassetid://77991697673856" })

InfoTab:AddSection({ "InformaÃ§Ãµes do Sistema" })
InfoTab:AddParagraph({ "VersÃ£o:", "High Menu v2.0 Fixed" })
InfoTab:AddParagraph({ "Jogo:", "Mini City RP" })
InfoTab:AddParagraph({ "Status:", "Sistema Otimizado" })

-- BotÃµes de servidor
InfoTab:AddButton({
    Name = "Reconectar ao Servidor",
    Callback = function()
        TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, State.player)
    end
})

InfoTab:AddButton({
    Name = "Buscar Novo Servidor",
    Callback = function()
        TeleportService:Teleport(game.PlaceId, State.player)
    end
})

-- Stats atualizadas
local statsLabel = InfoTab:AddParagraph({ "EstatÃ­sticas:", "Tempo: 0s | FPS: 60" })

spawn(function()
    while wait(1) do
        local elapsed = tick() - State.stats.startTime
        local fps = math.floor(1 / RunService.Heartbeat:Wait())
        
        statsLabel:Set("EstatÃ­sticas:", string.format(
            "Tempo: %ds | FPS: %d", 
            math.floor(elapsed), 
            fps
        ))
    end
end)

-- Sistema Auto CL (Quit quando vida baixa)
local autoCLActive = false

local function setupAutoCL(enabled)
    autoCLActive = enabled
    
    if enabled then
        -- FunÃ§Ã£o para monitorar vida
        local function monitorHealth()
            updateCharacter()
            if State.humanoid and autoCLActive then
                -- Conectar ao evento de mudanÃ§a de vida
                local healthConnection
                healthConnection = State.humanoid.HealthChanged:Connect(function(health)
                    if autoCLActive and State.humanoid then
                        local maxHealth = State.humanoid.MaxHealth
                        local healthPercentage = (health / maxHealth) * 100
                        
                        -- Se a vida estiver em 15% ou menos, sair do jogo
                        if healthPercentage <= 15 and health > 0 then
                            print("âš ï¸ Vida baixa detectada (" .. math.floor(healthPercentage) .. "%) - Ativando Auto CL...")
                            healthConnection:Disconnect()
                            State.player:Kick("Auto CL - Vida baixa detectada (" .. math.floor(healthPercentage) .. "%)")
                        end
                    end
                end)
                
                -- Armazenar conexÃ£o para limpeza posterior
                if not State.movement.connections.autoCL then
                    State.movement.connections.autoCL = {}
                end
                table.insert(State.movement.connections.autoCL, healthConnection)
            end
        end
        
        -- Monitorar personagem atual
        monitorHealth()
        
        -- Monitorar novos personagens (respawns)
        local charConnection = State.player.CharacterAdded:Connect(function()
            if autoCLActive then
                wait(2) -- Aguardar personagem carregar completamente
                monitorHealth()
            end
        end)
        
        -- Armazenar conexÃ£o para limpeza
        if not State.movement.connections.autoCL then
            State.movement.connections.autoCL = {}
        end
        table.insert(State.movement.connections.autoCL, charConnection)
        
        print("ðŸ”„ Auto CL ativado - Sair quando vida â‰¤ 15%")
    else
        -- Limpar todas as conexÃµes do Auto CL
        if State.movement.connections.autoCL then
            for _, connection in pairs(State.movement.connections.autoCL) do
                if connection then
                    connection:Disconnect()
                end
            end
            State.movement.connections.autoCL = {}
        end
        
        print("ðŸ”„ Auto CL desativado")
    end
end

-- Sistema Auto CL (Quit quando vida baixa)
local autoCLActive = false
local healthThreshold = 20 -- Valor absoluto de vida crÃ­tica

local function setupAutoCL(enabled)
    autoCLActive = enabled
    
    if enabled then
        -- FunÃ§Ã£o para monitorar vida
        local function monitorHealth()
            updateCharacter()
            if State.humanoid and autoCLActive then
                -- Conectar ao evento de mudanÃ§a de vida
                local healthConnection
                healthConnection = State.humanoid.HealthChanged:Connect(function(health)
                    if autoCLActive and State.humanoid and health > 0 then
                        -- Se a vida estiver no limite ou abaixo, sair IMEDIATAMENTE
                        if health <= healthThreshold then
                            print("âš ï¸ VIDA CRÃTICA! (" .. math.floor(health) .. " HP) - SAINDO AGORA!")
                            healthConnection:Disconnect()
                            
                            -- Sair super rÃ¡pido - sem delay
                            spawn(function()
                                State.player:Kick("Auto CL - Vida crÃ­tica! (" .. math.floor(health) .. " HP)")
                            end)
                        end
                    end
                end)
                
                -- VerificaÃ§Ã£o inicial da vida atual
                if State.humanoid.Health <= healthThreshold and State.humanoid.Health > 0 then
                    print("âš ï¸ VIDA JÃ CRÃTICA! (" .. math.floor(State.humanoid.Health) .. " HP) - SAINDO AGORA!")
                    healthConnection:Disconnect()
                    spawn(function()
                        State.player:Kick("Auto CL - Vida crÃ­tica! (" .. math.floor(State.humanoid.Health) .. " HP)")
                    end)
                end
                
                -- Armazenar conexÃ£o para limpeza posterior
                if not State.movement.connections.autoCL then
                    State.movement.connections.autoCL = {}
                end
                table.insert(State.movement.connections.autoCL, healthConnection)
            end
        end
        
        -- Monitorar personagem atual
        monitorHealth()
        
        -- Monitorar novos personagens (respawns)
        local charConnection = State.player.CharacterAdded:Connect(function()
            if autoCLActive then
                wait(1) -- Aguardar apenas 1 segundo para ativar mais rÃ¡pido
                monitorHealth()
            end
        end)
        
        -- Armazenar conexÃ£o para limpeza
        if not State.movement.connections.autoCL then
            State.movement.connections.autoCL = {}
        end
        table.insert(State.movement.connections.autoCL, charConnection)
        
        print("ðŸ”„ Auto CL ativado - Sair quando vida â‰¤ " .. healthThreshold .. " HP")
    else
        -- Limpar todas as conexÃµes do Auto CL
        if State.movement.connections.autoCL then
            for _, connection in pairs(State.movement.connections.autoCL) do
                if connection then
                    connection:Disconnect()
                end
            end
            State.movement.connections.autoCL = {}
        end
        
        print("ðŸ”„ Auto CL desativado")
    end
end

InfoTab:AddToggle({
    Name = "Auto CL",
    Default = false,
    Callback = setupAutoCL
})

InfoTab:AddSlider({
    Name = "Limite de Vida (HP)",
    Min = 5,
    Max = 80,
    Default = 20,
    Callback = function(value)
        healthThreshold = value
        print("ðŸ©º Limite de vida alterado para " .. value .. " HP")
    end
})

-- =================
-- ABA MOVEMENT SIMPLIFICADA
-- =================
local MovementTab = Window:MakeTab({ Title = "Movement", Icon = "rbxassetid://112065172702553" })

MovementTab:AddSection({ "Controles de Movimento" })

-- FunÃ§Ã£o Noclip
function activateNoclip(enabled)
    State.movement.noclipActive = enabled
    
    if enabled then
        State.movement.connections.noclip = RunService.Stepped:Connect(function()
            if State.character then
                for _, part in pairs(State.character:GetChildren()) do
                    if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                        part.CanCollide = false
                    end
                end
            end
        end)
    else
        if State.movement.connections.noclip then
            State.movement.connections.noclip:Disconnect()
            State.movement.connections.noclip = nil
        end
        
        if State.character then
            for _, part in pairs(State.character:GetChildren()) do
                if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                    part.CanCollide = true
                end
            end
        end
    end
end

MovementTab:AddToggle({
    Name = "Noclip Melhorado",
    Default = false,
    Callback = activateNoclip
})

-- FunÃ§Ã£o Pulo Infinito
function activateInfiniteJump(enabled)
    State.movement.infiniteJumpActive = enabled
    
    if enabled then
        State.movement.connections.jump = UserInputService.JumpRequest:Connect(function()
            if State.movement.infiniteJumpActive then
                updateCharacter()
                if State.humanoid then
                    State.humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                end
            end
        end)
    else
        if State.movement.connections.jump then
            State.movement.connections.jump:Disconnect()
            State.movement.connections.jump = nil
        end
    end
end

MovementTab:AddToggle({
    Name = "Pulo Infinito",
    Default = false,
    Callback = activateInfiniteJump
})

MovementTab:AddSlider({
    Name = "Velocidade Caminhada",
    Min = 16,
    Max = 500,
    Default = 16,
    Callback = function(value)
        State.settings.walkSpeed = value
        updateCharacter()
        if State.humanoid then
            State.humanoid.WalkSpeed = value
        end
    end
})

MovementTab:AddParagraph({
    "âš ï¸ AVISO IMPORTANTE:",
    "Use o pulo infinito com cuidado vocÃª pode acabar morrendo de dano de queda e use a velocidade de caminhada para ir mais rÃ¡pido usando o pulo infinito"
})

-- =================
-- SISTEMA FLY INTERNO (APENAS PARA FARM)
-- =================
function activateTweenFly(enabled)
    State.movement.flyActive = enabled
    
    if enabled then
        updateCharacter()
        if not State.rootPart then return end
        
        -- Criar BodyVelocity para controle suave
        State.movement.bodyVelocity = Instance.new("BodyVelocity")
        State.movement.bodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)
        State.movement.bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        State.movement.bodyVelocity.Parent = State.rootPart
        
        if State.humanoid then
            State.humanoid.PlatformStand = true
        end
        
        -- Sistema de controle suave com tween
        State.movement.connections.fly = RunService.Heartbeat:Connect(function()
            if not State.movement.flyActive or not State.rootPart then return end
            
            local camera = workspace.CurrentCamera
            local moveVector = Vector3.new(0, 0, 0)
            
            -- Detectar teclas pressionadas
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                moveVector = moveVector + camera.CFrame.LookVector
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                moveVector = moveVector - camera.CFrame.LookVector
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                moveVector = moveVector - camera.CFrame.RightVector
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                moveVector = moveVector + camera.CFrame.RightVector
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                moveVector = moveVector + Vector3.new(0, 1, 0)
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
                moveVector = moveVector - Vector3.new(0, 1, 0)
            end
            
            -- Aplicar velocidade com suavizaÃ§Ã£o
            local targetVelocity = moveVector * State.settings.flySpeed
            
            if State.movement.bodyVelocity then
                -- Usar tween para suavizar mudanÃ§as de velocidade
                local tweenInfo = TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
                local tween = TweenService:Create(
                    State.movement.bodyVelocity,
                    tweenInfo,
                    {Velocity = targetVelocity}
                )
                tween:Play()
            end
        end)
        
    else
        -- Limpar fly
        if State.movement.connections.fly then
            State.movement.connections.fly:Disconnect()
            State.movement.connections.fly = nil
        end
        
        if State.movement.bodyVelocity then
            State.movement.bodyVelocity:Destroy()
            State.movement.bodyVelocity = nil
        end
        
        if State.movement.flyTween then
            State.movement.flyTween:Cancel()
            State.movement.flyTween = nil
        end
        
        updateCharacter()
        if State.humanoid then
            State.humanoid.PlatformStand = false
        end
    end
end

-- =================
-- SISTEMA DE FARM OTIMIZADO
-- =================
local FarmTab = Window:MakeTab({ Title = "Farm Lixeiro", Icon = "rbxassetid://81812366414231" })

FarmTab:AddSection({ "Sistema de Coleta AutomÃ¡tica" })

-- FunÃ§Ã£o melhorada para encontrar lixos
local function findAllTrash()
    local trashList = {}
    local success, result = pcall(function()
        local mapFolder = workspace:FindFirstChild("MapaGeral")
        if not mapFolder then return {} end
        
        local gariFolder = mapFolder:FindFirstChild("Gari")
        if not gariFolder then return {} end
        
        local lixosFolder = gariFolder:FindFirstChild("Lixos")
        if not lixosFolder then return {} end
        
        for _, trash in pairs(lixosFolder:GetChildren()) do
            if trash:IsA("BasePart") then
                -- Buscar por diferentes variaÃ§Ãµes do nome
                local trashName = string.upper(trash.Name)
                if string.find(trashName, "LEXOS") or string.find(trashName, "LIXO") or string.find(trashName, "TRASH") then
                    local prompt = trash:FindFirstChildWhichIsA("ProximityPrompt")
                    if prompt and prompt.Enabled then
                        local distance = State.rootPart and (State.rootPart.Position - trash.Position).Magnitude or math.huge
                        table.insert(trashList, {part = trash, prompt = prompt, distance = distance})
                    end
                end
            end
        end
        
        -- Ordenar por distÃ¢ncia
        table.sort(trashList, function(a, b) return a.distance < b.distance end)
        
        return trashList
    end)
    
    return success and result or {}
end

-- FunÃ§Ã£o de movimento suave para farm
local function tweenToPosition(targetPosition, speed)
    updateCharacter()
    if not State.rootPart then return false end
    
    local distance = (State.rootPart.Position - targetPosition).Magnitude
    if distance < 5 then return true end
    
    local duration = distance / (speed or State.settings.flySpeed)
    duration = math.min(duration, 3) -- MÃ¡ximo 3 segundos por movimento
    
    local tweenInfo = TweenInfo.new(
        duration,
        Enum.EasingStyle.Quad,
        Enum.EasingDirection.Out,
        0, false, 0
    )
    
    local tween = TweenService:Create(State.rootPart, tweenInfo, {Position = targetPosition})
    
    local completed = false
    local startTime = tick()
    
    tween.Completed:Connect(function()
        completed = true
    end)
    
    tween:Play()
    
    -- Aguardar com timeout melhorado
    while not completed and State.farmActive and (tick() - startTime) < (duration + 1) do
        wait(0.05)
        -- Verificar se ainda estÃ¡ se movendo na direÃ§Ã£o certa
        if State.rootPart and (State.rootPart.Position - targetPosition).Magnitude < 8 then
            tween:Cancel()
            return true
        end
    end
    
    return completed or (State.rootPart and (State.rootPart.Position - targetPosition).Magnitude < 10)
end

-- Farm principal com melhorias
local function startTrashFarm()
    State.farmActive = true
    
    spawn(function()
        print("ðŸ—‘ï¸ Iniciando Farm Lixeiro...")
        
        -- Ativar fly temporariamente para farm
        local wasFlying = State.movement.flyActive
        if not wasFlying then
            activateTweenFly(true)
            wait(0.5) -- Aguardar fly ativar
        end
        
        while State.farmActive do
            updateCharacter()
            
            if State.rootPart and State.humanoid then
                local trashList = findAllTrash()
                
                if #trashList > 0 then
                    local target = trashList[1] -- Pegar o mais prÃ³ximo
                    local targetPos = target.part.Position + Vector3.new(0, 2, 0)
                    
                    -- Mover atÃ© o lixo
                    local success = tweenToPosition(targetPos, State.settings.flySpeed * 1.2)
                    
                    if success and State.farmActive then
                        -- Tentar coletar
                        wait(0.2)
                        if target.prompt and target.prompt.Enabled and target.prompt.Parent then
                            local attempts = 0
                            while attempts < 3 and target.prompt.Enabled and State.farmActive do
                                pcall(function()
                                    fireproximityprompt(target.prompt)
                                end)
                                attempts = attempts + 1
                                wait(0.1)
                            end
                            
                            State.stats.collected = State.stats.collected + 1
                            print("âœ… Coletado! Total:", State.stats.collected)
                        end
                    end
                    
                    wait(0.3) -- Pausa entre coletas
                else
                    print("ðŸ” Procurando lixos...")
                    wait(2) -- Aguardar respawn de lixos
                end
            else
                print("âŒ Erro: Personagem nÃ£o encontrado!")
                wait(2)
            end
            
            wait(0.1) -- Evitar lag
        end
        
        -- Desativar fly se nÃ£o estava ativo antes
        if not wasFlying then
            activateTweenFly(false)
        end
        
        print("ðŸ›‘ Farm Lixeiro parado!")
    end)
end

FarmTab:AddToggle({
    Name = "Farm Lixeiro AutomÃ¡tico",
    Default = false,
    Callback = function(enabled)
        if enabled then
            startTrashFarm()
        else
            State.farmActive = false
        end
    end
})

FarmTab:AddButton({
    Name = "Teleport para Lixos",
    Callback = function()
        local trashList = findAllTrash()
        if #trashList > 0 then
            updateCharacter()
            if State.rootPart then
                State.rootPart.Position = trashList[1].part.Position + Vector3.new(0, 5, 0)
                print("ðŸ“ Teleportado para Ã¡rea de lixos!")
            end
        else
            print("âŒ Nenhum lixo encontrado!")
        end
    end
})

-- =================
-- ESP MELHORADO
-- =================
local ESPTab = Window:MakeTab({ Title = "ESP", Icon = "rbxassetid://87627563993834" })

function updateESP(enabled)
    State.esp.active = enabled
    
    if enabled then
        -- Adicionar ESP para jogadores existentes
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= State.player and player.Character and not State.esp.highlights[player] then
                local highlight = Instance.new("Highlight")
                highlight.Name = "PlayerESP"
                highlight.FillTransparency = 0.7
                highlight.FillColor = Color3.fromRGB(0, 255, 0)
                highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                highlight.OutlineTransparency = 0
                highlight.Adornee = player.Character
                highlight.Parent = player.Character
                
                State.esp.highlights[player] = highlight
            end
        end
        
        -- Monitor para novos jogadores
        spawn(function()
            while State.esp.active do
                for _, player in pairs(Players:GetPlayers()) do
                    if player ~= State.player and player.Character and not State.esp.highlights[player] then
                        local highlight = Instance.new("Highlight")
                        highlight.Name = "PlayerESP"
                        highlight.FillTransparency = 0.7
                        highlight.FillColor = Color3.fromRGB(0, 255, 0)
                        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                        highlight.OutlineTransparency = 0
                        highlight.Adornee = player.Character
                        highlight.Parent = player.Character
                        
                        State.esp.highlights[player] = highlight
                    end
                end
                wait(1)
            end
        end)
    else
        -- Remover todos os highlights
        for player, highlight in pairs(State.esp.highlights) do
            if highlight then
                highlight:Destroy()
            end
            State.esp.highlights[player] = nil
        end
    end
end

ESPTab:AddToggle({
    Name = "Player ESP",
    Default = false,
    Callback = updateESP
})

ESPTab:AddToggle({
    Name = "Lixo ESP",
    Default = false,
    Callback = function(enabled)
        -- Implementar ESP para lixos se necessÃ¡rio
        if enabled then
            print("ðŸ—‘ï¸ Lixo ESP ativado!")
        else
            print("ðŸ—‘ï¸ Lixo ESP desativado!")
        end
    end
})

-- =================
-- SISTEMA DE LIMPEZA
-- =================
local function performCleanup()
    print("ðŸ§¹ Realizando limpeza do sistema...")
    
    -- Parar farm
    State.farmActive = false
    
    -- Desconectar todas as conexÃµes
    for _, connection in pairs(State.movement.connections) do
        if connection then
            connection:Disconnect()
        end
    end
    State.movement.connections = {}
    
    -- Limpar fly
    if State.movement.bodyVelocity then
        State.movement.bodyVelocity:Destroy()
        State.movement.bodyVelocity = nil
    end
    
    if State.movement.flyTween then
        State.movement.flyTween:Cancel()
        State.movement.flyTween = nil
    end
    
    -- Restaurar personagem
    updateCharacter()
    if State.humanoid then
        State.humanoid.WalkSpeed = 16
        State.humanoid.PlatformStand = false
    end
    
    -- Limpar ESP
    for player, highlight in pairs(State.esp.highlights) do
        if highlight then
            highlight:Destroy()
        end
    end
    State.esp.highlights = {}
    
    -- Restaurar colisÃµes
    if State.character then
        for _, part in pairs(State.character:GetChildren()) do
            if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                part.CanCollide = true
            end
        end
    end
    
    print("âœ… Limpeza concluÃ­da!")
end

-- Cleanup automÃ¡tico
Players.PlayerRemoving:Connect(function(player)
    if player == State.player then
        performCleanup()
    end
end)

-- NotificaÃ§Ã£o de carregamento
game:GetService("StarterGui"):SetCore("SendNotification", {
    Title = "High Menu",
    Text = "Sistema carregado com sucesso!",
    Duration = 3
})

print("âœ… High Menu v2.0 carregado com sucesso!")
print("ðŸš€ Fly com TweenService implementado!")
print("ðŸ“ˆ Sistema otimizado e estÃ¡vel!")
