-- Vision Hub para o jogo Kat e Arsenal com Aimbot para Facas e Tiros

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LP = Players.LocalPlayer
local mouse = LP:GetMouse()

-- Configurações do Aimbot e disparo
local Settings = {
    Enabled = false,
    AimPart = "Head",
    FOV = 100,
    Smoothness = 0.25,
    ShowFOV = true,
    Minimized = false,
    InstantAim = false,
    ShootOnClick = true, -- Disparar ao clicar na tela
    GameMode = "Kat" -- Pode ser "Kat" ou "Arsenal"
}

-- Função para identificar inimigos
local function IsEnemy(player)
    if LP.Team and player.Team and LP.Team == player.Team then
        return false
    end
    return true
end

-- Função para buscar o inimigo mais próximo
local function GetClosest()
    local closest, dist = nil, Settings.FOV
    local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and IsEnemy(plr) and plr.Character and plr.Character:FindFirstChild(Settings.AimPart) then
            local pos = plr.Character[Settings.AimPart].Position
            local screen, onScreen = Camera:WorldToViewportPoint(pos)
            if onScreen then
                local mag = (Vector2.new(screen.X, screen.Y) - center).Magnitude
                if mag < dist then
                    dist = mag
                    closest = plr
                end
            end
        end
    end
    return closest
end

-- Função de Disparo (Faca ou Tiro)
local function ShootAt(target)
    if target and target.Character and target.Character:FindFirstChild(Settings.AimPart) then
        -- Disparo no Jogo Kat (Faca)
        if Settings.GameMode == "Kat" then
            local knife = LP.Backpack:FindFirstChild("Knife")  -- Assume que a faca está no inventário
            if knife then
                -- Simula o lançamento da faca para o inimigo
                local direction = (target.Character[Settings.AimPart].Position - Camera.CFrame.Position).Unit
                local knifeClone = knife:Clone()
                knifeClone.Parent = workspace
                knifeClone.CFrame = Camera.CFrame
                knifeClone.Velocity = direction * 100  -- Ajuste a velocidade conforme necessário
            end
        -- Disparo no Jogo Arsenal (Arma de fogo)
        elseif Settings.GameMode == "Arsenal" then
            local gun = LP.Backpack:FindFirstChild("Gun")  -- Assume que a arma está no inventário
            if gun then
                -- Simula o disparo da arma
                local direction = (target.Character[Settings.AimPart].Position - Camera.CFrame.Position).Unit
                local bullet = Instance.new("Part")  -- Simula uma bala (substitua com o modelo correto)
                bullet.Size = Vector3.new(0.1, 0.1, 0.5)
                bullet.CFrame = Camera.CFrame
                bullet.Velocity = direction * 300  -- Ajuste a velocidade conforme necessário
                bullet.Parent = workspace
            end
        end
    end
end

-- Atualiza o ESP de todos os inimigos
RunService.RenderStepped:Connect(function()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and IsEnemy(plr) then
            DrawESP(plr)
        end
    end
end)

-- Loop do Aimbot (com suavização nova e modo instantâneo)
RunService.RenderStepped:Connect(function()
    if not Settings.Enabled then return end
    local target = GetClosest()
    if target and target.Character and target.Character:FindFirstChild(Settings.AimPart) then
        local aimPos = target.Character[Settings.AimPart].Position
        local camPos = Camera.CFrame.Position
        local direction = (aimPos - camPos).Unit
        local targetCFrame = CFrame.new(camPos, camPos + direction)

        if Settings.InstantAim then
            Camera.CFrame = targetCFrame
        else
            Camera.CFrame = Camera.CFrame:Lerp(targetCFrame, math.clamp(Settings.Smoothness, 0, 1))
        end
    end
end)

-- Função ESP para Inimigos
local function DrawESP(player)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local box = Drawing.new("Square")
        local nameTag = Drawing.new("Text")
        local distanceTag = Drawing.new("Text")
        local line = Drawing.new("Line")

        -- Configurações do box e texto
        box.Visible = true
        box.Color = Color3.fromRGB(255, 0, 0)  -- Cor da caixa (vermelho para inimigos)
        box.Thickness = 2
        box.Filled = false

        nameTag.Visible = true
        nameTag.TextColor3 = Color3.fromRGB(255, 255, 255)  -- Cor do nome
        nameTag.Font = Enum.Font.SourceSans
        nameTag.TextSize = 14

        distanceTag.Visible = true
        distanceTag.TextColor3 = Color3.fromRGB(255, 255, 255)  -- Cor da distância
        distanceTag.Font = Enum.Font.SourceSans
        distanceTag.TextSize = 14

        line.Visible = true
        line.Color = Color3.fromRGB(255, 0, 0)  -- Cor da linha
        line.Thickness = 2

        -- Atualiza o ESP durante o render
        RunService.RenderStepped:Connect(function()
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local rootPart = player.Character.HumanoidRootPart
                local screenPos, onScreen = Camera:WorldToViewportPoint(rootPart.Position)
                local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

                -- Verifica se o inimigo está na tela
                if onScreen then
                    local size = 40
                    local topLeft = Vector2.new(screenPos.X - size / 2, screenPos.Y - size / 2)
                    local bottomRight = Vector2.new(screenPos.X + size / 2, screenPos.Y + size / 2)

                    -- Atualiza a caixa
                    box.Position = topLeft
                    box.Size = UDim2.new(0, size, 0, size)

                    -- Atualiza o nome
                    nameTag.Text = player.Name
                    nameTag.Position = Vector2.new(screenPos.X, screenPos.Y - 25)

                    -- Atualiza a distância
                    local distance = math.floor((rootPart.Position - Camera.CFrame.Position).Magnitude)
                    distanceTag.Text = tostring(distance) .. "m"
                    distanceTag.Position = Vector2.new(screenPos.X, screenPos.Y + 25)

                    -- Atualiza a linha de conexão
                    line.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
                    line.To = Vector2.new(screenPos.X, screenPos.Y)
                else
                    -- Se o inimigo não estiver mais na tela, escondemos o ESP
                    box.Visible = false
                    nameTag.Visible = false
                    distanceTag.Visible = false
                    line.Visible = false
                end
            else
                -- Se o personagem foi destruído, escondemos o ESP
                box.Visible = false
                nameTag.Visible = false
                distanceTag.Visible = false
                line.Visible = false
            end
        end)
    end
end

-- GUI Principal
local gui = Instance.new("ScreenGui", LP:WaitForChild("PlayerGui"))
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 250, 0, 300)
frame.Position = UDim2.new(0.02, 0, 0.2, 0)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

-- Título
local title = Instance.new("TextLabel", frame)
title.Text = "Vision Hub"
title.Size = UDim2.new(1, -60, 0, 40)
title.Position = UDim2.new(0, 10, 0, 10)
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextXAlignment = Enum.TextXAlignment.Left

-- Botão Fechar
local close = Instance.new("TextButton", frame)
close.Text = "X"
close.Size = UDim2.new(0, 40, 0, 40)
close.Position = UDim2.new(1, -50, 0, 10)
close.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
close.TextColor3 = Color3.new(1,1,1)
close.Font = Enum.Font.SourceSansBold
close.TextSize = 18
close.MouseButton1Click:Connect(function()
    gui:Destroy()
end)

-- Botão Minimizar
local minimize = Instance.new("TextButton", frame)
minimize.Text = "-"
minimize.Size = UDim2.new(0, 40, 0, 40)
minimize.Position = UDim2.new(1, -90, 0, 10)
minimize.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
minimize.TextColor3 = Color3.new(1,1,1)
minimize.Font = Enum.Font.SourceSansBold
minimize.TextSize = 18
minimize.MouseButton1Click:Connect(function()
    Settings.Minimized = not Settings.Minimized
    for _, v in ipairs(frame:GetChildren()) do
        if v:IsA("TextButton") and v ~= close and v ~= minimize then
            v.Visible = not Settings.Minimized
        end
    end
end)

-- Função para criar Botões com mais estilo
local function createStyledButton(text, posY, callback)
    local btn = Instance.new("TextButton", frame)
    btn.Text = text
    btn.Size = UDim2.new(1, -20, 0, 40)
    btn.Position = UDim2.new(0, 10, 0, posY)
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 18
    btn.BorderSizePixel = 2
    btn.BorderColor3 = Color3.fromRGB(60, 60, 60)
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- Botões do Hub
local toggle = createStyledButton("Aimbot: OFF", 60, function()
    Settings.Enabled = not Settings.Enabled
    toggle.Text = "Aimbot: " .. (Settings.Enabled and "ON" or "OFF")
end)

local partBtn = createStyledButton("Parte: Head", 110, function()
    Settings.AimPart = Settings.AimPart == "Head" and "HumanoidRootPart" or "Head"
    partBtn.Text = "Parte: " .. Settings.AimPart
end)

local fovBtn = createStyledButton("FOV: 100", 160, function()
    Settings.FOV = Settings.FOV + 20
    if Settings.FOV > 200 then Settings.FOV = 40 end
    fovBtn.Text = "FOV: " .. Settings.FOV
end)

local smoothBtn = createStyledButton("Smooth: 0.25", 210, function()
    Settings.Smoothness = Settings.Smoothness + 0.05
    if Settings.Smoothness > 0.5 then Settings.Smoothness = 0.05 end
    smoothBtn.Text = "Smooth: " .. string.format("%.2f", Settings.Smoothness)
end)

local instantBtn = createStyledButton("Modo: Suave", 260, function()
    Settings.InstantAim = not Settings.InstantAim
    instantBtn.Text = Settings.InstantAim and "Modo: Instantâneo" or "Modo: Suave"
end)

-- Botões para selecionar o Modo do Jogo (Kat ou Arsenal)
local katBtn = createStyledButton("Modo: Kat", 310, function()
    Settings.GameMode = "Kat"
    katBtn.Text = "Modo: Kat"
end)

local arsenalBtn = createStyledButton("Modo: Arsenal", 360, function()
    Settings.GameMode = "Arsenal"
    arsenalBtn.Text = "Modo: Arsenal"
end)
