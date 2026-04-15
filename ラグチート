-- Steal a Brainrot トラップ系＋銃系 超高速使用切り替え (切り替えより早く使用) - Delta iOS対応
-- V: トラップ/銃スパムON | H: アンチラグ100000000× | F: サーバーラグ

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local NetworkClient = game:GetService("NetworkClient")
local Lighting = game:GetService("Lighting")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local lagActive = false
local lagIntensity = 0.65
local antiLagActive = true
local trapGunSpamActive = false

-- ==================== GUI ====================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "TrapGunCrasher"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = playerGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 250, 0, 290)
MainFrame.Position = UDim2.new(0.5, -125, 0.5, -145)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 20, 25)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 45)
TitleBar.BackgroundColor3 = Color3.fromRGB(220, 30, 30)
TitleBar.Parent = MainFrame
Instance.new("UICorner", TitleBar).CornerRadius = UDim.new(0, 12)

local TitleText = Instance.new("TextLabel")
TitleText.Size = UDim2.new(1, -100, 1, 0)
TitleText.Position = UDim2.new(0, 12, 0, 0)
TitleText.BackgroundTransparency = 1
TitleText.Text = "トラップ＋銃 超高速使用"
TitleText.TextColor3 = Color3.new(1,1,1)
TitleText.TextXAlignment = Enum.TextXAlignment.Left
TitleText.Font = Enum.Font.GothamBlack
TitleText.TextSize = 15
TitleText.Parent = TitleBar

-- ドラッグ機能
local dragging = false
local dragStart, startPos
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)
TitleBar.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = false end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- 右上トグルボタン
local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0, 50, 0, 50)
ToggleButton.Position = UDim2.new(1, -60, 0, 8)
ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 200, 255)
ToggleButton.Text = "≡"
ToggleButton.TextColor3 = Color3.new(1,1,1)
ToggleButton.TextSize = 28
ToggleButton.Parent = ScreenGui
Instance.new("UICorner", ToggleButton).CornerRadius = UDim.new(1, 0)

local uiVisible = true
ToggleButton.MouseButton1Click:Connect(function()
    uiVisible = not uiVisible
    MainFrame.Visible = uiVisible
end)

local function CreateButton(text, yPos, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.9, 0, 0, 40)
    btn.Position = UDim2.new(0.05, 0, 0, yPos)
    btn.BackgroundColor3 = Color3.fromRGB(55, 60, 70)
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 14
    btn.Parent = MainFrame
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    btn.MouseButton1Click:Connect(callback)
    return btn
end

CreateButton("サーバーラグ: ON/OFF (F)", 60, function() lagActive = not lagActive; print("サーバーラグ: "..(lagActive and "ON" or "OFF")) end)
CreateButton("アンチラグ 100000000×: ON/OFF (H)", 105, function()
    antiLagActive = not antiLagActive
    if antiLagActive then
        settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
        Lighting.GlobalShadows = false
        Lighting.Brightness = 0
        Lighting.FogEnd = 999999
        settings().Physics.PhysicsEnvironmentalThrottle = Enum.EnviromentalPhysicsThrottle.Disabled
        settings().Physics.AllowSleep = true
        
        for _, obj in pairs(workspace:GetDescendants()) do
            if obj:IsA("BasePart") then
                obj.Material = Enum.Material.SmoothPlastic
                obj.CastShadow = false
                obj.Transparency = 0.6
            elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") then
                obj.Enabled = false
            end
        end
        print("アンチラグ 100000000× ON")
    else
        settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
        Lighting.GlobalShadows = true
        print("アンチラグ OFF")
    end
end)

CreateButton("トラップ＋銃 超高速使用: ON/OFF (V)", 150, function()
    trapGunSpamActive = not trapGunSpamActive
    print("トラップ＋銃スパム: " .. (trapGunSpamActive and "ON（切り替えより早く使用）" or "OFF"))
end)

-- ==================== ロジック ====================
-- サーバーラグ
RunService.Heartbeat:Connect(function()
    pcall(function()
        if lagActive then
            NetworkClient:SetOutgoingKBPSLimit(lagIntensity * 1000)
        else
            NetworkClient:SetOutgoingKBPSLimit(0)
        end
    end)
end)

-- トラップ系＋銃系のみ 超高速Equip → 使用 → Unequip
local targetTools = {"Trap", "Bee Launcher", "Taser Gun", "Paintball Gun", "Gun"}  -- 銃系・トラップ系を追加（必要に応じて名前調整）
local currentIndex = 1
local lastAction = 0

RunService.Heartbeat:Connect(function()
    if not trapGunSpamActive then return end
    if tick() - lastAction < 0.001 then return end  -- 極限高速（使用を切り替えより早く）

    local char = player.Character
    if not char then return end
    local backpack = player:FindFirstChild("Backpack")
    if not backpack then return end

    -- 対象ツールだけ抽出
    local toolList = {}
    for _, item in ipairs(backpack:GetChildren()) do
        if item:IsA("Tool") and table.find(targetTools, item.Name) then
            table.insert(toolList, item)
        end
    end
    for _, item in ipairs(char:GetChildren()) do
        if item:IsA("Tool") and table.find(targetTools, item.Name) then
            table.insert(toolList, item)
        end
    end

    if #toolList == 0 then return end

    local tool = toolList[currentIndex]
    if tool then
        if tool.Parent ~= char then
            tool.Parent = char  -- Equip
            wait(0.0005)        -- 微小待機で使用を確実に
            if tool:FindFirstChild("Activate") or tool:FindFirstChildWhichIsA("BindableEvent") then
                pcall(function() tool:Activate() end)  -- 使用（Activate）
            end
        else
            tool.Parent = backpack  -- Unequip
        end
    end

    currentIndex = (currentIndex % #toolList) + 1
    lastAction = tick()
end)

-- キー入力
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F then
        lagActive = not lagActive
        print("サーバーラグ: " .. (lagActive and "ON" or "OFF"))
    elseif input.KeyCode == Enum.KeyCode.H then
        antiLagActive = not antiLagActive
        -- アンチラグ処理（上と同じ）
        if antiLagActive then
            settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
            Lighting.GlobalShadows = false
            Lighting.Brightness = 0
            settings().Physics.PhysicsEnvironmentalThrottle = Enum.EnviromentalPhysicsThrottle.Disabled
        else
            settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
            Lighting.GlobalShadows = true
        end
    elseif input.KeyCode == Enum.KeyCode.V then
        trapGunSpamActive = not trapGunSpamActive
        print("トラップ＋銃超高速使用: " .. (trapGunSpamActive and "ON" or "OFF"))
    end
end)

player.CharacterAdded:Connect(function()
    wait(1)
    if antiLagActive then
        settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
        Lighting.GlobalShadows = false
        Lighting.Brightness = 0
        settings().Physics.PhysicsEnvironmentalThrottle = Enum.EnviromentalPhysicsThrottle.Disabled
    end
end)

print("=== トラップ系＋銃系 超高速使用切り替え Loaded ===")
print("使い方：")
print("1. Hキーでアンチラグ100000000×をON（必須！）")
print("2. BackpackにTrap、Bee Launcher、Taser Gun、Paintball Gunなどを入れる")
print("3. VキーでスパムON → 切り替えより早く使用連打")
print("短時間（1〜3秒）だけONにして即OFF！ 自分ラグが出たらアンチラグを再ON")
