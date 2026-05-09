-- [[ R-HUB AETHER | OMNI-REMASTERED v3.5.0 ]] --

local TARGET_KEY = "six seven"
local player = game:GetService("Players").LocalPlayer
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

-- ระบบจัดการภายใน (Core State)
local State = {
    Version = 1.00,
    IsExpanded = false,
    NextWarp = 20,
    Authenticated = false,
    SpecialCooldown = 180 -- 3 นาที สำหรับปุ่มแดงแถบดำ
}

-- [[ 1. UI CONSTRUCTION ]] --
local ScreenGui = Instance.new("ScreenGui", player.PlayerGui)
ScreenGui.Name = "RH_Omni_Final"
ScreenGui.IgnoreGuiInset = true

local Island = Instance.new("Frame", ScreenGui)
Island.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Island.Position = UDim2.new(0.5, -80, 0, 15)
Island.Size = UDim2.new(0, 160, 0, 36)
Island.ClipsDescendants = true
local IslandCorner = Instance.new("UICorner", Island)
IslandCorner.CornerRadius = UDim.new(0, 18)

local UIStroke = Instance.new("UIStroke", Island)
UIStroke.Color = Color3.fromRGB(255, 255, 255)
UIStroke.Thickness = 2
UIStroke.Transparency = 0.4

local Header = Instance.new("TextLabel", Island)
Header.Size = UDim2.new(1, 0, 0, 36)
Header.BackgroundTransparency = 1
Header.Font = Enum.Font.GothamBold
Header.TextColor3 = Color3.new(1, 1, 1)
Header.TextSize = 10
Header.Text = "AETHER OMNI v1.00"

local Content = Instance.new("CanvasGroup", Island)
Content.Size = UDim2.new(1, 0, 1, -36)
Content.Position = UDim2.new(0, 0, 0, 36)
Content.BackgroundTransparency = 1
Content.GroupTransparency = 1
Content.Visible = false

local UIList = Instance.new("UIListLayout", Content)
UIList.Padding = UDim.new(0, 8)
UIList.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- [[ 2. STATS & INTERFACE ]] --
local function createStat(text, color)
    local l = Instance.new("TextLabel", Content)
    l.Size = UDim2.new(0.85, 0, 0, 18)
    l.BackgroundTransparency = 1
    l.Font = Enum.Font.GothamMedium
    l.TextColor3 = color
    l.TextSize = 10
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Text = text
    return l
end

local WinLabel = createStat("🏆 Wins: 0", Color3.fromRGB(0, 255, 150))
local TimerLabel = createStat("🕒 Next Warp: 20s", Color3.fromRGB(200, 200, 200))
local SpecialLabel = createStat("💎 Special: Waiting...", Color3.fromRGB(255, 100, 100))

local KeyBox = Instance.new("TextBox", Content)
KeyBox.Size = UDim2.new(0.85, 0, 0, 30)
KeyBox.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
KeyBox.Font = Enum.Font.GothamBold
KeyBox.PlaceholderText = "ENTER KEY"
KeyBox.Text = ""
KeyBox.TextColor3 = Color3.new(1, 1, 1)
KeyBox.TextSize = 10
Instance.new("UICorner", KeyBox)

-- [[ 3. SMART LOGIC (Warp & Security) ]] --
local function getSpecialTarget()
    -- ค้นหาปุ่มสีแดงแถบดำ (จากไฟล์ สกรีนช็อต 2026-05-09 121606.png)
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") and v.Color == Color3.fromRGB(255, 0, 0) then
            local p = v.Parent
            local isTrap = p.Name:lower():find("teleport") or v.Name:lower():find("teleport")
            local hasBlack = false
            for _, c in pairs(p:GetChildren()) do
                if c:IsA("BasePart") and c.Color == Color3.fromRGB(0,0,0) then hasBlack = true break end
            end
            if hasBlack and not isTrap then return v end
        end
    end
    return nil
end

-- [[ 4. ANIMATION & CONTROL ]] --
local function toggleIsland(expand)
    State.IsExpanded = expand
    local targetSize = expand and UDim2.new(0, 220, 0, 180) or UDim2.new(0, 160, 0, 36)
    local targetPos = expand and UDim2.new(0.5, -110, 0, 15) or UDim2.new(0.5, -80, 0, 15)
    
    TweenService:Create(Island, TweenInfo.new(0.6, Enum.EasingStyle.Elastic, Enum.EasingDirection.Out), {
        Size = targetSize,
        Position = targetPos
    }):Play()
    
    if expand then
        Content.Visible = true
        TweenService:Create(Content, TweenInfo.new(0.3), {GroupTransparency = 0}):Play()
    else
        TweenService:Create(Content, TweenInfo.new(0.2), {GroupTransparency = 1}):Play()
        task.delay(0.2, function() if not State.IsExpanded then Content.Visible = false end end)
    end
end

-- ระบบตรวจสอบคีย์
KeyBox.FocusLost:Connect(function(enter)
    if enter then
        if KeyBox.Text == TARGET_KEY then
            State.Authenticated = true
            State.Version = 2.00
            Header.Text = "AETHER OMNI v2.00"
            UIStroke.Color = Color3.fromRGB(255, 215, 0)
            KeyBox.Text = "ACCESS GRANTED"
            task.wait(1)
            KeyBox.Visible = false
        else
            KeyBox.Text = ""; KeyBox.PlaceholderText = "WRONG KEY!"
            for i = 1, 6 do -- อนิเมชั่นสั่นเมื่อผิด
                Island.Position = Island.Position + UDim2.new(0, (i%2==0 and 5 or -5), 0, 0)
                task.wait(0.05)
            end
        end
    end
end)

-- Loop การทำงานหลัก
task.spawn(function()
    local lastSpecial = 0
    while true do
        for i = 20, 1, -1 do
            State.NextWarp = i
            task.wait(1)
        end
        
        local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        if root then
            -- 1. วาร์ปปกติ
            local finish = workspace:FindFirstChild("Finish") or workspace:FindFirstChild("Win")
            if finish then
                root.Velocity = Vector3.zero
                root.CFrame = finish.CFrame * CFrame.new(0, 3, 0)
            end
            
            -- 2. วาร์ปพิเศษ (ถ้าปลดล็อก v2.00 และครบ 3 นาที)
            if State.Authenticated and (os.time() - lastSpecial) >= State.SpecialCooldown then
                task.wait(1)
                local sp = getSpecialTarget()
                if sp then
                    root.CFrame = sp.CFrame * CFrame.new(0, 3, 0)
                    lastSpecial = os.time()
                end
            end
        end
    end
end)

-- อัปเดตข้อมูล UI Real-time
RunService.Heartbeat:Connect(function()
    if State.IsExpanded then
        local wins = player:FindFirstChild("leaderstats") and player.leaderstats:FindFirstChild("Wins")
        WinLabel.Text = "🏆 Total Wins: " .. (wins and wins.Value or 0)
        TimerLabel.Text = "🕒 Next Warp: " .. State.NextWarp .. "s"
        
        if State.Authenticated then
            SpecialLabel.Text = "💎 Special: Ready"
            SpecialLabel.TextColor3 = Color3.fromRGB(0, 200, 255)
        else
            SpecialLabel.Text = "💎 Special: v2.00 Required"
        end
    end
end)

-- เริ่มต้นใช้งาน
Island.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        toggleIsland(not State.IsExpanded)
    end
end)

task.wait(0.5)
toggleIsland(true) -- เปิดมาให้กรอกคีย์ทันที
