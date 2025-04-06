-- Carregar Fluent UI
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

-- Criar Janela
local Window = Fluent:CreateWindow({
    Title = "BloxFruits ✅ 1.1.0",
    SubTitle = "by GuizinhoGamers0",
    TabWidth = 160,
    Size = UDim2.fromOffset(520, 300),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Options = Fluent.Options

-- TAB: Auto Farm
local AutoFarmTab = Window:AddTab({ Title = "Auto Farm", Icon = "hammer" })
local AutoFarmAtivo = false

AutoFarmTab:AddToggle("AutoFarmToggle", { Title = "Auto farm nível", Default = false }):OnChanged(function(state)
    AutoFarmAtivo = state
    if state then
        task.spawn(function()
            while AutoFarmAtivo and task.wait(0.5) do
                pcall(function()
                    local player = game.Players.LocalPlayer
                    local char = player.Character or player.CharacterAdded:Wait()
                    local hrp = char:FindFirstChild("HumanoidRootPart")
                    local humanoid = char:FindFirstChildOfClass("Humanoid")
                    if not hrp or not humanoid then return end
                    local closestEnemy, minDist = nil, math.huge
                    for _,v in pairs(workspace.Enemies:GetChildren()) do
                        if v:FindFirstChild("HumanoidRootPart") and v:FindFirstChildOfClass("Humanoid") and v:FindFirstChildOfClass("Humanoid").Health > 0 then
                            local dist = (v.HumanoidRootPart.Position - hrp.Position).Magnitude
                            if dist < minDist then
                                minDist = dist
                                closestEnemy = v
                            end
                        end
                    end
                    if closestEnemy then
                        hrp.CFrame = closestEnemy.HumanoidRootPart.CFrame * CFrame.new(0, 15, 0)
                        local vim = game:GetService("VirtualInputManager")
                        vim:SendMouseButtonEvent(0, 0, 0, true, game, 1)
                        task.wait()
                        vim:SendMouseButtonEvent(0, 0, 0, false, game, 1)
                    end
                end)
            end
        end)
    end
end)

-- TAB: Combat PvP
local CombatTab = Window:AddTab({ Title = "Combat PvP", Icon = "swords" })
local selectedPlayer = nil
local playersDropdown = CombatTab:AddDropdown("PlayersList", {
    Title = "Escolher jogador",
    Values = {},
    Multi = false,
    Default = nil
})

CombatTab:AddToggle("AttackPlayerToggle", { Title = "Atacar jogador", Default = false }):OnChanged(function(state)
    if state and selectedPlayer then
        task.spawn(function()
            local player = game.Players.LocalPlayer
            local char = player.Character or player.CharacterAdded:Wait()
            local hrp = char:WaitForChild("HumanoidRootPart")
            while Options.AttackPlayerToggle.Value and task.wait(0.5) do
                local target = game.Players:FindFirstChild(selectedPlayer)
                if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
                    hrp.CFrame = target.Character.HumanoidRootPart.CFrame * CFrame.new(0, 15, 0)
                    local vim = game:GetService("VirtualInputManager")
                    vim:SendMouseButtonEvent(0, 0, 0, true, game, 1)
                    task.wait()
                    vim:SendMouseButtonEvent(0, 0, 0, false, game, 1)
                end
            end
        end)
    end
end)

playersDropdown:OnChanged(function(value)
    selectedPlayer = value
end)

local function atualizarJogadores()
    local nomes = {}
    for _,v in pairs(game.Players:GetPlayers()) do
        if v ~= game.Players.LocalPlayer then
            table.insert(nomes, v.Name)
        end
    end
    playersDropdown:SetValues(nomes)
end
atualizarJogadores()
game.Players.PlayerAdded:Connect(atualizarJogadores)
game.Players.PlayerRemoving:Connect(atualizarJogadores)

-- TAB: Visual
local VisualTab = Window:AddTab({ Title = "Visual", Icon = "eye" })

-- Função de ESPs com controle
local ESPs = {
    Frutas = {},
    Jogadores = {}
}

local function criarESP(obj, nome, lista)
    if obj and obj:IsA("BasePart") then
        local gui = Instance.new("BillboardGui", obj)
        gui.Name = "ESP"
        gui.Size = UDim2.new(0, 100, 0, 40)
        gui.AlwaysOnTop = true
        gui.Adornee = obj
        local label = Instance.new("TextLabel", gui)
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.Text = nome
        label.TextColor3 = Color3.new(1, 1, 1)
        label.TextStrokeTransparency = 0
        table.insert(ESPs[lista], gui)
    end
end

local function limparESP(lista)
    for _,v in pairs(ESPs[lista]) do
        if v and v.Parent then
            v:Destroy()
        end
    end
    ESPs[lista] = {}
end

-- ESP Frutas
VisualTab:AddToggle("ESPFrutas", { Title = "Ver Frutas", Default = false }):OnChanged(function(state)
    limparESP("Frutas")
    if state then
        for _,v in pairs(game.Workspace:GetDescendants()) do
            if v:IsA("Tool") and v:FindFirstChild("Handle") then
                criarESP(v.Handle, v.Name, "Frutas")
            end
        end
    end
end)

-- ESP Jogadores
VisualTab:AddToggle("ESPPlayers", { Title = "Ver Jogadores", Default = false }):OnChanged(function(state)
    limparESP("Jogadores")
    if state then
        for _,p in pairs(game.Players:GetPlayers()) do
            if p ~= game.Players.LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                criarESP(p.Character.HumanoidRootPart, p.Name, "Jogadores")
            end
        end
    end
end)

-- TAB: Configuração
local ConfigTab = Window:AddTab({ Title = "Configuração", Icon = "settings" })
ConfigTab:AddButton({ Title = "Todos os códigos", Callback = function()
    for _,v in pairs(require(game.ReplicatedStorage.Util.CameraShaker).ActiveRigs) do
        game:GetService("ReplicatedStorage").Remotes.Redeem:InvokeServer(v)
    end
end })

-- Gerenciadores
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)
SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({})
InterfaceManager:SetFolder("FluentScriptHub")
SaveManager:SetFolder("FluentScriptHub/BloxFruits")
SaveManager:BuildConfigSection(ConfigTab)
SaveManager:LoadAutoloadConfig()
