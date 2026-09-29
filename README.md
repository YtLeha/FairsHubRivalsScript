# FairsHubRivalsScript
FairsHubRivals
-- StarterPlayer/StarterPlayerScripts/FairsHubClient.client.lua
local Players = game:GetService("Players")
local RS = game:GetService("ReplicatedStorage")
local UIS = game:GetService("UserInputService")

local Remotes = RS:WaitForChild("Remotes")
local plr = Players.LocalPlayer
local playerGui = plr:WaitForChild("PlayerGui")

-- === Создаём GUI ===
local screen = Instance.new("ScreenGui")
screen.Name = "FairsHub"
screen.ResetOnSpawn = false
screen.Parent = playerGui

-- Главная кнопка открытия
local openBtn = Instance.new("TextButton")
openBtn.Size = UDim2.new(0, 120, 0, 40)
openBtn.Position = UDim2.new(0, 20, 0.5, -20)
openBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
openBtn.TextColor3 = Color3.fromRGB(120, 220, 255)
openBtn.Font = Enum.Font.GothamBold
openBtn.TextSize = 16
openBtn.Text = "🎯 FairsHub"
openBtn.Parent = screen

local corner1 = Instance.new("UICorner"); corner1.CornerRadius = UDim.new(0, 8); corner1.Parent = openBtn

-- Основное окно
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 320, 0, 380)
frame.Position = UDim2.new(0.5, -160, 0.5, -190)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
frame.Visible = false
frame.Parent = screen

local corner2 = Instance.new("UICorner"); corner2.CornerRadius = UDim.new(0, 12); corner2.Parent = frame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 45)
title.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
title.TextColor3 = Color3.fromRGB(120, 220, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.Text = "FairsHub — магазин читов"
title.Parent = frame
local corner3 = Instance.new("UICorner"); corner3.CornerRadius = UDim.new(0, 12); corner3.Parent = title

local coinsLabel = Instance.new("TextLabel")
coinsLabel.Size = UDim2.new(1, -20, 0, 30)
coinsLabel.Position = UDim2.new(0, 10, 0, 50)
coinsLabel.BackgroundTransparency = 1
coinsLabel.TextColor3 = Color3.fromRGB(255, 220, 100)
coinsLabel.Font = Enum.Font.GothamBold
coinsLabel.TextSize = 16
coinsLabel.TextXAlignment = Enum.TextXAlignment.Left
coinsLabel.Text = "💰 Монеты: 200"
coinsLabel.Parent = frame

-- Список способностей
local abilities = {
    { key = "XRay",       name = "Wallhack (X-Ray)",  price = 50 },
    { key = "AutoAim",    name = "Auto-Aim",          price = 80 },
    { key = "AutoDodge",  name = "Auto-Dodge",        price = 60 },
    { key = "SpeedBoost", name = "Speed Boost",       price = 40 },
}

local y = 90
for _, ab in ipairs(abilities) do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -20, 0, 50)
    btn.Position = UDim2.new(0, 10, 0, y)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    btn.TextColor3 = Color3.fromRGB(220, 220, 255)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 15
    btn.Text = ab.name.."  •  "..ab.price.."💰"
    btn.Parent = frame
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, 8); c.Parent = btn

    btn.MouseButton1Click:Connect(function()
        Remotes.BuyAbility:FireServer(ab.key)
    end)

    -- Правая кнопка — использовать
    btn.MouseButton2Click:Connect(function()
        Remotes.UseAbility:FireServer(ab.key)
    end)

    y += 60
end

local hint = Instance.new("TextLabel")
hint.Size = UDim2.new(1, -20, 0, 30)
hint.Position = UDim2.new(0, 10, 1, -40)
hint.BackgroundTransparency = 1
hint.TextColor3 = Color3.fromRGB(150, 150, 180)
hint.Font = Enum.Font.Gotham
hint.TextSize = 12
hint.Text = "ЛКМ — купить  •  ПКМ — использовать"
hint.Parent = frame

-- Открытие/закрытие
openBtn.MouseButton1Click:Connect(function()
    frame.Visible = not frame.Visible
end)

UIS.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F then
        frame.Visible = not frame.Visible
    end
end)

-- Уведомления
local notifyContainer = Instance.new("Frame")
notifyContainer.Size = UDim2.new(0, 300, 1, -20)
notifyContainer.Position = UDim2.new(1, -320, 0, 10)
notifyContainer.BackgroundTransparency = 1
notifyContainer.Parent = screen

local notifyLayout = Instance.new("UIListLayout")
notifyLayout.Padding = UDim.new(0, 6)
notifyLayout.SortOrder = Enum.SortOrder.LayoutOrder
notifyLayout.Parent = notifyContainer

Remotes.Notify.OnClientEvent:Connect(function(text, color)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, 0, 0, 36)
    lbl.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    lbl.TextColor3 = color
    lbl.Font = Enum.Font.Gotham
    lbl.TextSize = 14
    lbl.Text = text
    lbl.Parent = notifyContainer
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, 6); c.Parent = lbl

    task.delay(4, function()
        if lbl then lbl:Destroy() end
    end)
end)

-- Голосование за бан (ПКМ по игроку в списке — упрощённо)
-- Здесь для примера: голос через кнопку G по ближайшему игроку
UIS.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.G then
        local char = plr.Character
        if not char then return end
        local closest, dist = nil, 50
        for _, other in Players:GetPlayers() do
            if other ~= plr and other.Character and other.Character:FindFirstChild("Head") then
                local d = (other.Character.Head.Position - char.Head.Position).Magnitude
                if d < dist then closest, dist = other, d end
            end
        end
        if closest then
            Remotes.VoteBan:FireServer(closest.UserId)
        end
    end
end)
