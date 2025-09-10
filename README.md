-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local localPlayer = Players.LocalPlayer

-- Variáveis do Anti-AFK
local antiAFKEnabled = false
local jumpInterval = 60 -- padrão 60s
local lastJump = 0

-- Função para obter o humanoid atual (suporta respawn)
local function getHumanoid()
    local character = localPlayer.Character
    if character then
        return character:FindFirstChildOfClass("Humanoid")
    end
    return nil
end

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AntiAFK_GUI"
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

-- Botão ligar/desligar
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 120, 0, 40)
toggleButton.Position = UDim2.new(0, 20, 0, 100)
toggleButton.BackgroundColor3 = Color3.fromRGB(255,0,0)
toggleButton.Text = "OFF"
toggleButton.TextColor3 = Color3.new(1,1,1)
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextScaled = true
toggleButton.Parent = screenGui

-- Label do contador
local counterLabel = Instance.new("TextLabel")
counterLabel.Size = UDim2.new(0, 120, 0, 30)
counterLabel.Position = UDim2.new(0, 20, 0, 150)
counterLabel.BackgroundTransparency = 1
counterLabel.TextColor3 = Color3.new(1,1,1)
counterLabel.Font = Enum.Font.SourceSansBold
counterLabel.TextScaled = true
counterLabel.Text = "Próximo salto: 60s"
counterLabel.Parent = screenGui

-- Label para alterar intervalo
local intervalLabel = Instance.new("TextLabel")
intervalLabel.Size = UDim2.new(0, 120, 0, 30)
intervalLabel.Position = UDim2.new(0, 20, 0, 190)
intervalLabel.BackgroundTransparency = 1
intervalLabel.TextColor3 = Color3.new(1,1,0)
intervalLabel.Font = Enum.Font.SourceSansBold
intervalLabel.TextScaled = true
intervalLabel.Text = "Intervalo: 60s"
intervalLabel.Parent = screenGui

-- Atualiza visual do botão
local function updateButton()
    if antiAFKEnabled then
        toggleButton.BackgroundColor3 = Color3.fromRGB(0,255,0)
        toggleButton.Text = "ON"
    else
        toggleButton.BackgroundColor3 = Color3.fromRGB(255,0,0)
        toggleButton.Text = "OFF"
    end
end

-- Alternar ON/OFF
toggleButton.MouseButton1Click:Connect(function()
    antiAFKEnabled = not antiAFKEnabled
    updateButton()
end)

-- Atalho F
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F then
        antiAFKEnabled = not antiAFKEnabled
        updateButton()
        print("[Anti-AFK] ", antiAFKEnabled and "Ligado" or "Desligado")
    end
end)

-- Permitir alterar intervalo clicando no label
intervalLabel.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local newInterval = tonumber(game:GetService("Players").LocalPlayer:PromptInput("Digite intervalo de salto (segundos):", tostring(jumpInterval)))
        if newInterval and newInterval > 0 then
            jumpInterval = newInterval
            intervalLabel.Text = "Intervalo: "..jumpInterval.."s"
        end
    end
end)

-- Loop principal
RunService.RenderStepped:Connect(function(delta)
    if not antiAFKEnabled then return end
    lastJump = lastJump + delta
    local humanoid = getHumanoid()
    if humanoid then
        local timeLeft = math.max(0, math.ceil(jumpInterval - lastJump))
        counterLabel.Text = "Próximo salto: "..timeLeft.."s"
        if lastJump >= jumpInterval then
            lastJump = 0
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end
end)

-- Inicializa
updateButton()
