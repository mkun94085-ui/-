-- [[ R-HUB AETHER | OMNI-REMASTERED v3.6.0 - WARP FIX ]] --

local TARGET_KEY = "six seven"
local player = game:GetService("Players").LocalPlayer
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local State = {
    Version = 1.00,
    IsExpanded = false,
    NextWarp = 15, -- ปรับเวลาให้ไวขึ้นเพื่อทดสอบ
    Authenticated = false
}

-- [[ 1. SMART WIN FINDER (แก้ไขจุดไม่วาร์ป) ]] --
local function findTheWin()
    -- วิธีที่ 1: หาจากชื่อมาตรฐาน
    local target = workspace:FindFirstChild("Finish", true) or workspace:FindFirstChild("Win", true) or workspace:FindFirstChild("End", true)
    
    -- วิธีที่ 2: ถ้าไม่เจอ ให้หาจาก Object ที่มี TouchInterest (ปุ่มที่แตะแล้วได้รางวัล)
    if not target then
        for _, v in pairs(workspace:GetDescendants()) do
            if v:IsA("TouchTransmitter") and (v.Parent.Name:lower():find("win") or v.Parent.Name:lower():find("finish")) then
                target = v.Parent
                break
            end
        end
    end
    
    return target
end

-- [[ 2. UI CONSTRUCTION (WITH ANIMATION) ]] --
local ScreenGui = Instance.new("ScreenGui", player.PlayerGui)
local Island = Instance.new("Frame", ScreenGui)
Island.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Island.Position = UDim2.new(0.5, -80, 0, 15)
Island.Size = UDim2.new(0, 160, 0, 36)
Island.ClipsDescendants = true
Instance.new("UICorner", Island).CornerRadius = UDim.new(0, 18)
local UIStroke = Instance.new("UIStroke", Island)
UIStroke.Color = Color3.new(1, 1, 1)
UIStroke.Thickness = 2

local Header = Instance.new("TextLabel", Island)
Header.Size = UDim2.new(1, 0, 0, 36)
Header.BackgroundTransparency = 1; Header.Font = Enum.Font.GothamBold
Header.TextColor3 = Color3.new(1, 1, 1); Header.TextSize = 10
Header.Text = "AETHER FIX v1.00"

local Content = Instance.new("CanvasGroup", Island)
Content.Size = UDim2.new(1, 0, 1, -36); Content.Position = UDim2.new(0, 0, 0, 36)
Content.BackgroundTransparency = 1; Content.GroupTransparency = 1; Content.Visible = false
local UIList = Instance.new("UIListLayout", Content)
UIList.Padding = UDim.new(0, 8); UIList.HorizontalAlignment = Enum.HorizontalAlignment.Center

local WinLabel = Instance.new("TextLabel", Content)
WinLabel.Size = UDim2.new(0.85, 0, 0, 18); WinLabel.BackgroundTransparency = 1
WinLabel.Font = Enum.Font.GothamMedium; WinLabel.TextColor3 = Color3.fromRGB(0, 255, 150)
WinLabel.TextSize = 10; WinLabel.Text = "🏆 Wins: Loading..."

local TimerLabel = Instance.new("TextLabel", Content)
TimerLabel.Size = UDim2.new(0.85, 0, 0, 18); TimerLabel.BackgroundTransparency = 1
TimerLabel.Font = Enum.Font.GothamMedium; TimerLabel.TextColor3 = Color3.new(0.8, 0.8, 0.8)
TimerLabel.TextSize = 10; TimerLabel.Text = "🕒 Warp: 15s"

local KeyBox = Instance.new("TextBox", Content)
KeyBox.Size = UDim2.new(0.85, 0, 0, 30); KeyBox.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
KeyBox.Font = Enum.Font.GothamBold; KeyBox.PlaceholderText = "ENTER KEY"; KeyBox.Text = ""
KeyBox.TextColor3 = Color3.new(1, 1, 1); KeyBox.TextSize = 10
Instance.new("UICorner", KeyBox)

-- [[ 3. CORE WARP ENGINE (แก้ไขระบบวาร์ป) ]] --
task.spawn(function()
    while true do
        for i = State.NextWarp, 1, -1 do
            State.NextWarp = i
            task.wait(1)
        end
        
        local char = player.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        
        if root then
            local winPoint = findTheWin()
            if winPoint then
                -- แจ้งสถานะบน UI
                Header.Text = "🌀 WARPING..."
                
                -- ล้างความเร็วตัวละคร (กันบัค)
                root.Velocity = Vector3.new(0, 0, 0)
                
                -- วาร์ป
                if winPoint:IsA("Model") then
                    root.CFrame = winPoint:GetModelCFrame() * CFrame.new(0, 3, 0)
                else
                    root.CFrame = winPoint.CFrame * CFrame.new(0, 3, 0)
                end
                
                -- ล็อกตัวชั่วคราวเพื่อให้เกมโหลดทัน
                root.Anchored = true
                task.wait(0.5)
                root.Anchored = false
                Header.Text = "AETHER v" .. string.format("%.2f", State.Version)
            else
                warn("Aether: Could not find win point!")
                Header.Text = "⚠️ WIN NOT FOUND"
                task.wait(2)
            end
        end
        State.NextWarp = 15 -- Reset loop
    end
end)

-- [[ 4. INTERACTION & AUTH ]] --
local function toggle(expand)
    State.IsExpanded = expand
    local targetSize = expand and UDim2.new(0, 200, 0, 160) or UDim2.new(0, 160, 0, 36)
    TweenService:Create(Island, TweenInfo.new(0.6, Enum.EasingStyle.Elastic), {Size = targetSize}):Play()
    Content.Visible = true
    TweenService:Create(Content, TweenInfo.new(0.3), {GroupTransparency = expand and 0 or 1}):Play()
end

KeyBox.FocusLost:Connect(function(enter)
    if enter and KeyBox.Text == TARGET_KEY then
        State.Authenticated = true; State.Version = 2.00
        UIStroke.Color = Color3.fromRGB(255, 215, 0)
        Header.Text = "AETHER v2.00 PREMIUM"
        KeyBox.Visible = false
    end
end)

RunService.Heartbeat:Connect(function()
    if State.IsExpanded then
        local w = player:FindFirstChild("leaderstats") and player.leaderstats:FindFirstChild("Wins")
        WinLabel.Text = "🏆 Wins: " .. (w and w.Value or 0)
        TimerLabel.Text = "🕒 Warp: " .. State.NextWarp .. "s"
    end
end)

Island.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then toggle(not State.IsExpanded) end end)

-- รันครั้งแรกให้ขยายรอเลย
task.wait(0.5)
toggle(true)
