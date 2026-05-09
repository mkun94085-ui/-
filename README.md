-- [[ R-HUB AETHER | v2.00 - CLEAN KEY INTERFACE ]] --

local TARGET_KEY = "six seven"
local ADMIN_KEY = "Mark"
local player = game:GetService("Players").LocalPlayer
local TweenService = game:GetService("TweenService")

local currentVer = "1.00" -- เริ่มต้นที่เวอร์ชันฟรี
local isExpanded = false

-- [[ 1. UI DESIGN - แก้ไขข้อความบัง ]] --
local ScreenGui = Instance.new("ScreenGui", player.PlayerGui)
ScreenGui.Name = "RH_Clean_UI"

local Island = Instance.new("Frame", ScreenGui)
Island.BackgroundColor3 = Color3.new(0, 0, 0)
Island.Size = UDim2.new(0, 160, 0, 35)
Island.Position = UDim2.new(0.5, -80, 0, 15)
Island.ClipsDescendants = true -- ป้องกันข้อความล้น
local IslandCorner = Instance.new("UICorner", Island)
IslandCorner.CornerRadius = UDim.new(1, 0)

local MainLabel = Instance.new("TextLabel", Island)
MainLabel.Size = UDim2.new(1, 0, 1, 0)
MainLabel.BackgroundTransparency = 1
MainLabel.Font = Enum.Font.GothamBold
MainLabel.TextColor3 = Color3.new(1, 1, 1)
MainLabel.TextSize = 10
MainLabel.Text = "AETHER v1.00 [FREE]" -- แสดงสถานะฟรีชัดเจน

-- [[ 2. SMART WARP ENGINE ]] --
local function getSpecialButton()
    -- ค้นหาปุ่มสีแดงแถบดำตามภาพ สกรีนช็อต 2026-05-09 121606.png
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") and v.Color == Color3.fromRGB(255, 0, 0) then
            local p = v.Parent
            local hasBlack = false
            for _, c in pairs(p:GetChildren()) do
                if c:IsA("BasePart") and (c.Color == Color3.fromRGB(0,0,0) or c.Name:lower():find("base")) then
                    hasBlack = true
                end
            end
            -- ห้ามวาร์ปไปปุ่ม Teleport เด็ดขาด
            if hasBlack and not p.Name:lower():find("teleport") then
                return v
            end
        end
    end
    return nil
end

-- [[ 3. VERSION UPGRADE LOGIC ]] --
local function activateUpgrade(ver)
    currentVer = ver
    Island.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    MainLabel.Text = "EVOLVING TO v" .. ver .. "..."
    task.wait(1)
    
    if ver == "2.00" then
        -- ระบบ Status Cycle ของ v1.10 และ ฟังก์ชันปุ่มแดงของ v2.00
        task.spawn(function()
            local cycle = {"SYSTEM: OPTIMIZED", "v2.00 ULTIMATE", "SPECIAL WARP: READY"}
            while true do
                for _, txt in pairs(cycle) do
                    if not isExpanded then MainLabel.Text = txt end
                    task.wait(3)
                end
            end
        end)
    end
end

-- [[ 4. CLEAN KEY INPUT - แก้ไขปัญหา UI บังกัน ]] --
local function openKeyCenter()
    -- ซ่อนข้อความหลักก่อนแสดงช่องกรอก
    MainLabel.TextTransparency = 1
    
    local KeyInput = Instance.new("TextBox", Island)
    KeyInput.Size = UDim2.new(1, -20, 1, 0)
    KeyInput.Position = UDim2.new(0, 10, 0, 0)
    KeyInput.BackgroundTransparency = 1
    KeyInput.Font = Enum.Font.GothamBold
    KeyInput.TextColor3 = Color3.fromRGB(0, 255, 150)
    KeyInput.PlaceholderText = "TYPE KEY HERE"
    KeyInput.Text = ""
    KeyInput.TextSize = 10
    
    KeyInput.FocusLost:Connect(function(enter)
        if enter then
            if KeyInput.Text == TARGET_KEY or KeyInput.Text == ADMIN_KEY then
                KeyInput:Destroy()
                MainLabel.TextTransparency = 0
                activateUpgrade("2.00")
            else
                KeyInput.Text = ""
                KeyInput.PlaceholderText = "WRONG KEY"
                task.wait(1)
                KeyInput:Destroy()
                MainLabel.TextTransparency = 0
                MainLabel.Text = "AETHER v1.00 [FREE]"
            end
        else
            KeyInput:Destroy()
            MainLabel.TextTransparency = 0
        end
    end)
end

-- [[ 5. INITIALIZE ]] --
Island.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        if currentVer == "1.00" then
            openKeyCenter() -- ถ้ายังไม่เติม จะเปิดหน้ากรอกคีย์
        else
            isExpanded = not isExpanded -- ถ้าเติมแล้ว จะขยายดูสถานะ
            Island:TweenSize(isExpanded and UDim2.new(0, 220, 0, 80) or UDim2.new(0, 160, 0, 35), "Out", "Back", 0.3)
        end
    end
end)

-- Start Basic Farm (Free Version)
task.spawn(function()
    while true do
        local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        if root then
            -- วาร์ปปกติ (v1.00)
            local finish = workspace:FindFirstChild("Finish") or workspace:FindFirstChild("Win")
            if finish then
                root.CFrame = finish.CFrame * CFrame.new(0, 3, 0)
            end
            
            -- วาร์ปปุ่มพิเศษ (v2.00 เท่านั้น)
            if currentVer == "2.00" then
                task.wait(2) -- เว้นจังหวะ
                local spBtn = getSpecialButton()
                if spBtn then root.CFrame = spBtn.CFrame * CFrame.new(0, 3, 0) end
            end
        end
        task.wait(20)
    end
end)
