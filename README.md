function CriarInterfaceFluent()
    local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
    local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
    local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

    local Window = Fluent:CreateWindow({
        Title = "BloxFruits ☑️ " .. Fluent.Version,
        SubTitle = "by GuizinhoGamers0",
        TabWidth = 160,
        Size = UDim2.fromOffset(500, 270),
        Acrylic = true,
        Theme = "Dark",
        MinimizeKey = Enum.KeyCode.LeftControl
    })

    -- Tabs com ícones válidos do Lucide
    local FarmTab = Window:AddTab({ Title = "Auto Farm", Icon = "hammer" }) -- picareta/martelo
    local PvPTab = Window:AddTab({ Title = "Combat PvP", Icon = "swords" }) -- espadas cruzadas
    local ConfigTab = Window:AddTab({ Title = "Configuração", Icon = "settings" }) -- engrenagem

    local Options = Fluent.Options
    local AutoFarmAtivo = false
    local JogadorSelecionado = nil

    -- AUTO FARM
    FarmTab:AddToggle("AutoFarmToggle", { Title = "Auto farm nível", Default = false }):OnChanged(function(state)
        AutoFarmAtivo = state

        if AutoFarmAtivo then
            task.spawn(function()
                while AutoFarmAtivo and task.wait(0.5) do
                    pcall(function()
                        local player = game.Players.LocalPlayer
                        local character = player.Character or player.CharacterAdded:Wait()
                        local rootPart = character:FindFirstChild("HumanoidRootPart")

                        local closestEnemy, minDistance = nil, math.huge
                        for _, v in pairs(workspace.Enemies:GetChildren()) do
                            if v:FindFirstChild("HumanoidRootPart") and v:FindFirstChildOfClass("Humanoid") and v:FindFirstChildOfClass("Humanoid").Health > 0 then
                                local distance = (v.HumanoidRootPart.Position - rootPart.Position).Magnitude
                                if distance < minDistance then
                                    closestEnemy = v
                                    minDistance = distance
                                end
                            end
                        end

                        if closestEnemy then
                            rootPart.CFrame = closestEnemy.HumanoidRootPart.CFrame * CFrame.new(0, 0, 2)
                            local VirtualInputManager = game:GetService("VirtualInputManager")
                            VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 1)
                            task.wait()
                            VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 1)
                        end
                    end)
                end
            end)
        end
    end)

    -- COMBAT PVP
    local listaJogadores = {}
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer then
            table.insert(listaJogadores, player.Name)
        end
    end

    PvPTab:AddDropdown("ListaJogadores", {
        Title = "Selecionar Jogador",
        Values = listaJogadores,
        Multi = false,
        Default = nil
    }):OnChanged(function(value)
        JogadorSelecionado = value
    end)

    local AtacarJogadorAtivo = false

    PvPTab:AddToggle("ToggleAtacarJogador", {
        Title = "Atacar Jogador",
        Description = "Ataca automaticamente o jogador selecionado",
        Default = false
    }):OnChanged(function(state)
        AtacarJogadorAtivo = state

        if AtacarJogadorAtivo then
            task.spawn(function()
                while AtacarJogadorAtivo and task.wait(0.3) do
                    pcall(function()
                        if JogadorSelecionado then
                            local alvo = game.Players:FindFirstChild(JogadorSelecionado)
                            if alvo and alvo.Character and alvo.Character:FindFirstChild("HumanoidRootPart") then
                                local root = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                                if root then
                                    root.CFrame = alvo.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 2)
                                    local VirtualInputManager = game:GetService("VirtualInputManager")
                                    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 1)
                                    task.wait()
                                    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 1)
                                end
                            end
                        end
                    end)
                end
            end)
        end
    end)

    -- CONFIGURAÇÕES - RESGATAR CÓDIGOS
    ConfigTab:AddButton({
        Title = "Todos os Códigos",
        Description = "Resgata todos os códigos ativos do Blox Fruits",
        Callback = function()
            local codes = {
                "kittgaming", "Sub2Fer999", "Enyu_is_Pro", "Magicbus", "JCWK", "Starcodeheo",
                "Bluxxy", "fudd10_v2", "fudd10", "Bignews", "Sub2CaptainMaui", "SUB2GAMERROBOT_RESET1",
                "SUB2GAMERROBOT_EXP1", "Sub2NoobMaster123", "Sub2UncleKizaru", "Sub2Daigrock",
                "Axiore", "TantaiGaming", "StrawHatMaine"
            }

            for _, code in pairs(codes) do
                pcall(function()
                    game:GetService("ReplicatedStorage").Remotes.Redeem:InvokeServer(code)
                end)
            end
        end
    })

    -- ADDONS
    SaveManager:SetLibrary(Fluent)
    InterfaceManager:SetLibrary(Fluent)
    SaveManager:IgnoreThemeSettings()
    SaveManager:SetIgnoreIndexes({})
    InterfaceManager:SetFolder("FluentScriptHub")
    SaveManager:SetFolder("FluentScriptHub/specific-game")
    SaveManager:LoadAutoloadConfig()
end

-- Executa automaticamente
CriarInterfaceFluent()
