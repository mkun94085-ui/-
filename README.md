-- [[ R-HUB AETHER | OMNI-EXPANSION v3.0.0 ]] --

local TARGET_KEY = "six seven"
local player = game:GetService("Players").LocalPlayer
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

-- Kernel State
local RH_Kernel = {
    Version = 1.00,
    AccessLevel = "FREE",
    IsExpanded = false,
    NextWarp = 20,
    StartTime = os.time()
}

-- [[ 1. UI CONSTRUCTION (EXPANDABLE) ]] --
local ScreenGui = Instance.new("ScreenGui", player.PlayerGui)
ScreenGui.Name = "RH_Omni_V3"
ScreenGui.IgnoreGuiInset = true

local Island = Instance.new("Frame", ScreenGui)
Island.BackgroundColor3 = Color3.new(0, 0, 0)
Island.Position = UDim2.new(0.5, -80, 0, 15)
Island.Size = UDim2.new(0, 160, 0, 36)
Island.ClipsDescendants = true
local IslandCorner = Instance.new("UICorner", Island)
IslandCorner.CornerRadius = UDim.new(0, 18)

local UIStroke = Instance.new("UIStroke", Island)
UIStroke.Color = Color3.fromRGB(255, 255, 255)
UIStroke.Thickness = 1.5

-- หัวข้อตอนย่อ (Compact Mode)
local CompactLabel = Instance.new("TextLabel", Island)
CompactLabel.Size = UDim2.new(1, 0, 0, 36)
CompactLabel.BackgroundTransparency = 1
CompactLabel.Font = Enum.Font.GothamBold
CompactLabel.TextColor3 = Color3.new(1, 1, 1)
CompactLabel.TextSize = 10
CompactLabel.Text = "AETHER v" .. string.format("%.2f", RH_Kernel.Version)

-- แผงควบคุมตอนขยาย (Expanded Content)
local ContentFrame = Instance.new("Frame", Island)
ContentFrame.Size = UDim2.new(1, 0, 1, -36)
ContentFrame.Position = UDim2.new(0, 0, 0, 36)
ContentFrame.BackgroundTransparency = 1
ContentFrame.Visible = false

local UIList = Instance.new("UIListLayout", ContentFrame)
UIList.Padding = UDim.new(0, 5)
UIList.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- ฟังก์ชันสร้างข้อความในแผง
local function createStatLabel(name)
    local lbl = Instance.new("TextLabel", ContentFrame)
    lbl.Size = UDim2.new(0.9, 0, 0, 15)
    lbl.BackgroundTransparency = 1
    lbl.Font = Enum.Font.GothamMedium
    lbl.TextColor3 = Color3.fromRGB(200, 200, 200)
    lbl.TextSize = 9
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    return lbl
end

local WinLabel = createStatLabel("Wins")
local TimerLabel = createStatLabel("Next Warp")
local RealTimeLabel = createStatLabel("Real Time")

-- ช่องกรอกคีย์ (Key Input ในแผงขยาย)
local KeyBox = Instance.new("TextBox", ContentFrame)
KeyBox.Size = UDim2.new(0.85, 0, 0, 25)
KeyBox.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
KeyBox.Font = Enum.Font.GothamBold
KeyBox.PlaceholderText = "ENTER KEY TO UPDATE"
KeyBox.Text = ""
KeyBox.TextColor3 = Color3.new(1, 1, 1)
KeyBox.TextSize = 9
Instance.new("UICorner", KeyBox)

-- [[ 2. LOGIC & UPDATES ]] --

-- ระบบอัปเดตข้อมูล Real-time
RunService.RenderStepped:Connect(function()
    if RH_Kernel.IsExpanded then
        -- 1. Wins
        local wins = player:FindFirstChild("leaderstats") and player.leaderstats:FindFirstChild("Wins")
        WinLabel.Text = "🏆 Wins: " .. (wins and wins.Value or 0)
        
        -- 2. Timer
        TimerLabel.Text = "🕒 Next Warp: " .. RH_Kernel.NextWarp .. "s"
        
        -- 3. Real Time
        RealTimeLabel.Text = "📅 Clock: " .. os.date("%X")
    end
end)

-- ระบบวาร์ป (Core Engine)
task.spawn(function()
    while true do
        for i = 20, 1, -1 do
            RH_Kernel.NextWarp = i
            task.wait(1)
        end
        
        local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        if root then
            -- วาร์ปพื้นฐาน
            local finish = workspace:FindFirstChild("Finish") or workspace:FindFirstChild("Win")
            if finish then
                root.CFrame = finish.CFrame * CFrame.new(0, 3, 0)
                root.Anchored = true; task.wait(0.5); root.Anchored = false
            end
            
            -- วาร์ปพิเศษ (v2.00+)
            if RH_Kernel.Version >= 2.00 then
                -- (ระบบสแกนปุ่มแดงแถบดำทำงานตรงนี้)
            end
        end
    end
end)

-- ระบบปลดล็อกคีย์
KeyBox.FocusLost:Connect(function(enter)
    if enter then
        if KeyBox.Text == TARGET_KEY then
            RH_Kernel.Version = 2.00
            RH_Kernel.AccessLevel = "PREMIUM"
            UIStroke.Color = Color3.fromRGB(255, 200, 0)
            CompactLabel.Text = "AETHER v2.00 [ULTIMATE]"
            KeyBox.Text = "SUCCESS!"
            task.wait(1)
            KeyBox.PlaceholderText = "v2.00 ACTIVE"
            KeyBox.Text = ""
        else
            KeyBox.Text = "INVALID KEY"
            task.wait(1)
            KeyBox.Text = ""
        end
    end
end)

-- [[ 3. INTERACTION (EXPAND/COLLAPSE) ]] --
Island.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        RH_Kernel.IsExpanded = not RH_Kernel.IsExpanded
        
        local targetSize = RH_Kernel.IsExpanded and UDim2.new(0, 200, 0, 160) or UDim2.new(0, 160, 0, 36)
        local targetPos = RH_Kernel.IsExpanded and UDim2.new(0.5, -100, 0, 15) or UDim2.new(0.5, -80, 0, 15)
        
        ContentFrame.Visible = RH_Kernel.IsExpanded
        CompactLabel.TextTransparency = RH_Kernel.IsExpanded and 0.5 or 0
        
        TweenService:Create(Island, TweenInfo.new(0.5, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
            Size = targetSize,
            Position = targetPos
        }):Play()
    end
end)
