-- =============================================
-- ПОЛНЫЙ СКРИПТ MM2 CANDY FARM + GUI
-- =============================================

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local VirtualInputManager = game:GetService("VirtualInputManager")
local LocalPlayer = Players.LocalPlayer

-- Глобальные настройки
getgenv().AutoFarm = false
getgenv().AntiAFK = false
getgenv().FarmSpeed = 0.5

-- СОЗДАНИЕ GUI
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local AutoFarmBtn = Instance.new("TextButton")
local AntiAFKBtn = Instance.new("TextButton")
local Title = Instance.new("TextLabel")
local Status = Instance.new("TextLabel")

ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

-- Основной фрейм
MainFrame.Parent = ScreenGui
MainFrame.Size = UDim2.new(0, 220, 0, 180)
MainFrame.Position = UDim2.new(0, 10, 0, 10)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
MainFrame.Active = true
MainFrame.Draggable = true

-- Заголовок
Title.Parent = MainFrame
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
Title.Text = "🎮 MM2 Auto Farm"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 16

-- Статус
Status.Parent = MainFrame
Status.Size = UDim2.new(1, 0, 0, 20)
Status.Position = UDim2.new(0, 0, 0.85, 0)
Status.BackgroundTransparency = 1
Status.Text = "Готов к работе!"
Status.TextColor3 = Color3.fromRGB(200, 200, 200)
Status.TextSize = 12

-- Кнопка авто-фарма
AutoFarmBtn.Parent = MainFrame
AutoFarmBtn.Size = UDim2.new(0.8, 0, 0, 45)
AutoFarmBtn.Position = UDim2.new(0.1, 0, 0.2, 0)
AutoFarmBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
AutoFarmBtn.Text = "🚜 Авто-фарм: ВЫКЛ"
AutoFarmBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
AutoFarmBtn.TextSize = 14

-- Кнопка анти-АФК
AntiAFKBtn.Parent = MainFrame
AntiAFKBtn.Size = UDim2.new(0.8, 0, 0, 45)
AntiAFKBtn.Position = UDim2.new(0.1, 0, 0.55, 0)
AntiAFKBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
AntiAFKBtn.Text = "⏰ Анти-АФК: ВЫКЛ"
AntiAFKBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
AntiAFKBtn.TextSize = 14

-- ФУНКЦИИ GUI
AutoFarmBtn.MouseButton1Click:Connect(function()
    getgenv().AutoFarm = not getgenv().AutoFarm
    if getgenv().AutoFarm then
        AutoFarmBtn.Text = "🚜 Авто-фарм: ВКЛ"
        AutoFarmBtn.TextColor3 = Color3.fromRGB(100, 255, 100)
        Status.Text = "Авто-фарм активирован!"
        startFarm()
    else
        AutoFarmBtn.Text = "🚜 Авто-фарм: ВЫКЛ"
        AutoFarmBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
        Status.Text = "Авто-фарм остановлен"
    end
end)

AntiAFKBtn.MouseButton1Click:Connect(function()
    getgenv().AntiAFK = not getgenv().AntiAFK
    if getgenv().AntiAFK then
        AntiAFKBtn.Text = "⏰ Анти-АФК: ВКЛ"
        AntiAFKBtn.TextColor3 = Color3.fromRGB(100, 255, 100)
        Status.Text = "Анти-АФК активирован!"
        startAntiAFK()
    else
        AntiAFKBtn.Text = "⏰ Анти-АФК: ВЫКЛ"
        AntiAFKBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
        Status.Text = "Анти-АФК выключен"
    end
end)

-- ФУНКЦИИ ФАРМА
function notify(msg)
    Status.Text = msg
    wait(3)
    if getgenv().AutoFarm then
        Status.Text = "Фармим конфеты..."
    else
        Status.Text = "Готов к работе"
    end
end

function isAlive()
    return LocalPlayer.Character and 
           LocalPlayer.Character:FindFirstChild("Humanoid") and 
           LocalPlayer.Character.Humanoid.Health > 0
end

function findCandies()
    local candies = {}
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("Part") and (obj.Name:lower():find("candy") or obj.Name:lower():find("halloween")) then
            if isAlive() then
                local distance = (LocalPlayer.Character.HumanoidRootPart.Position - obj.Position).Magnitude
                if distance < 200 then
                    table.insert(candies, obj)
                end
            end
        end
    end
    return candies
end

function collectCandy(candy)
    if not isAlive() then return false end
    
    -- Телепорт к конфете
    LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(candy.Position + Vector3.new(0, 3, 0))
    wait(0.3)
    
    -- Сбор конфеты
    if candy and candy.Parent then
        firetouchinterest(LocalPlayer.Character.HumanoidRootPart, candy, 0)
        wait(0.1)
        firetouchinterest(LocalPlayer.Character.HumanoidRootPart, candy, 1)
        return true
    end
    return false
end

function startFarm()
    spawn(function()
        while getgenv().AutoFarm do
            if not isAlive() then
                notify("Персонаж мертв, ждем респавн...")
                LocalPlayer.CharacterAdded:Wait()
                wait(3)
            end
            
            local candies = findCandies()
            if #candies > 0 then
                local collected = 0
                for i, candy in pairs(candies) do
                    if not getgenv().AutoFarm then break end
                    if collectCandy(candy) then
                        collected = collected + 1
                    end
                    wait(getgenv().FarmSpeed)
                end
                notify("Собрано конфет: " .. collected)
            else
                -- Двигаемся случайно если конфет нет
                local randomPos = Vector3.new(
                    math.random(-50, 50),
                    20,
                    math.random(-50, 50)
                )
                if isAlive() then
                    LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(randomPos)
                end
                notify("Ищем конфеты...")
            end
            
            wait(1)
        end
    end)
end

-- ФУНКЦИЯ АНТИ-АФК
function startAntiAFK()
    spawn(function()
        while getgenv().AntiAFK do
            if isAlive() then
                -- Движение вперед-назад
                VirtualInputManager:SendKeyEvent(true, "W", false, game)
                wait(0.5)
                VirtualInputManager:SendKeyEvent(false, "W", false, game)
                wait(0.5)
                VirtualInputManager:SendKeyEvent(true, "S", false, game)
                wait(0.5)
                VirtualInputManager:SendKeyEvent(false, "S", false, game)
                
                -- Поворот камеры
                local cam = workspace.CurrentCamera
                if cam then
                    cam.CFrame = cam.CFrame * CFrame.Angles(0, math.rad(45), 0)
                end
            end
            wait(25) -- Ждем 25 секунд
        end
    end)
end

-- АВТО-РЕСПАВН
LocalPlayer.CharacterAdded:Connect(function()
    wait(3)
    if getgenv().AutoFarm then
        notify("Персонаж зареспавнился!")
    end
end)

-- УВЕДОМЛЕНИЕ О ЗАГРУЗКЕ
notify("Скрипт загружен! 🎮")

print("🎮 MM2 Auto Farm загружен!")
print("✅ GUI создано")
print("✅ Авто-фарм готов")
print("✅ Анти-АФК готов")
