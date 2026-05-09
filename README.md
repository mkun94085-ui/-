-- [[ R-HUB ULTIMATE | V1.3.0 -> ROAD TO 10.01 ]] --

local TARGET_KEY = "six seven"
local SPECIAL_KEY = "Mark"

local player = game:GetService("Players").LocalPlayer
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

-- ระบบจัดการเวอร์ชัน (สามารถขยับได้ถึง 10.01)
local VersionData = {
    Current = 1.30,
    Max = 10.01,
    Status = "Stable"
}

local isExpanded = false
local isWarping = false

-- [[ 1. UI DESIGN ]] --
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RHUB_Evolution_System"
ScreenGui.Parent = player:WaitForChild("PlayerGui")
ScreenGui.IgnoreGuiInset = true

local Island = Instance.new("TextButton")
Island.Parent = ScreenGui
Island.BackgroundColor3 = Color3.new(0, 0, 0)
Island.Position = UDim2.new(0.5, -75, 0, 15)
Island.Size = UDim2.new(0, 150, 0, 32)
Island.AutoButtonColor = false
Island.Text = ""
Island.ClipsDescendants = true
local IslandCorner = Instance.new("UICorner", Island)
IslandCorner.CornerRadius = UDim.new(1, 0)

local MainLabel = Instance.new("TextLabel")
MainLabel.Parent = Island
MainLabel.Size = UDim2.new(1, 0, 1, 0)
MainLabel.BackgroundTransparency = 1
MainLabel.Font = Enum.Font.GothamBold
MainLabel.TextColor3 = Color3.new(1, 1, 1)
MainLabel.TextSize = 11
MainLabel.Text = "R-HUB v" .. string.format("%.2f", VersionData.Current)

-- Notification Panel (เด้งอัปเดตแก้ไขบัค)
local UpdateBox = Instance.new("Frame")
UpdateBox.Parent = ScreenGui
UpdateBox.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
UpdateBox.Position = UDim2.new(1, 50, 0.4, 0)
UpdateBox.Size = UDim2.new(0, 250, 0, 120)
Instance.new("UICorner", UpdateBox)

local NotifTitle = Instance.new("TextLabel")
NotifTitle.Parent = UpdateBox
NotifTitle.Size = UDim2.new(1, 0, 0.3, 0)
NotifTitle.BackgroundTransparency = 1
NotifTitle.Font = Enum.Font.GothamBold
NotifTitle.TextColor3 = Color3.fromRGB(0, 255, 150)
NotifTitle.Text = "BUG FIX DETECTED!"
NotifTitle.TextSize = 14

local NotifDesc = Instance.new("TextLabel")
NotifDesc.Parent = UpdateBox
NotifDesc.Position = UDim2.new(0, 0, 0.3, 0)
NotifDesc.Size = UDim2.new(1, 0, 0.3, 0)
NotifDesc.BackgroundTransparency = 1
NotifDesc.Font = Enum.Font.Gotham
NotifDesc.TextColor3 = Color3.new(0.9, 0.9, 0.9)
NotifDesc.Text = "Optimizing Warp Logic...\nTarget Version: 1.31"
NotifDesc.TextSize = 10

local UpBtn = Instance.new("TextButton")
UpBtn.Parent = UpdateBox
UpBtn.Position = UDim2.new(0.08, 0, 0.65, 0)
UpBtn.Size = UDim2.new(0, 100, 0, 30)
UpBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
UpBtn.Font = Enum.Font.GothamBold
UpBtn.Text = "UPDATE NOW"
UpBtn.TextColor3 = Color3.new(1, 1, 1)
UpBtn.TextSize = 11
Instance.new("UICorner", UpBtn)

local SkipBtn = Instance.new("TextButton")
SkipBtn.Parent = UpdateBox
SkipBtn.Position = UDim2.new(0.55, 0, 0.65, 0)
SkipBtn.Size = UDim2.new(0, 100, 0, 30)
SkipBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
SkipBtn.Font = Enum.Font.GothamBold
SkipBtn.Text = "IGNORE"
SkipBtn.TextColor3 = Color3.new(1, 1, 1)
SkipBtn.TextSize = 11
Instance.new("UICorner", SkipBtn)

-- [[ 2. WARP SYSTEM (STABLE 20S) ]] --
local function getFinishCFrame()
    local target = workspace:FindFirstChild("Finish") or workspace:FindFirstChild("Win")
    if not target then
        for _, v in pairs(workspace:GetDescendants()) do
            if v:IsA("TouchTransmitter") and (v.Parent.Name:lower():find("win") or v.Parent.Name:lower():find("finish")) then
                target = v.Parent break
            end
        end
    end
    return target and target.CFrame
end

local function stableWarp()
    if isWarping then return end
    isWarping = true
    pcall(function()
        local char = player.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local finishCF = getFinishCFrame()
        if root and finishCF then
            root.Velocity = Vector3.zero
            root.CFrame = finishCF * CFrame.new(0, 3, 0)
            root.Anchored = true
            task.wait(0.6) -- เพิ่มเวลาล็อกตัวเล็กน้อยเพื่อความเสถียร
            root.Anchored = false
        end
    end)
    isWarping = false
end

-- [[ 3. UPDATE LOGIC ]] --
local function applyUpdate()
    if VersionData.Current < VersionData.Max then
        VersionData.Current = VersionData.Current + 0.01
        MainLabel.Text = "UPDATING..."
        task.wait(2)
        MainLabel.Text = "v" .. string.format("%.2f", VersionData.Current) .. " ACTIVE"
        UpdateBox:TweenPosition(UDim2.new(1, 50, 0.4, 0), "In", "Sine", 0.5)
        task.wait(1)
    end
end

UpBtn.MouseButton1Click:Connect(applyUpdate)
SkipBtn.MouseButton1Click:Connect(function()
    UpdateBox:TweenPosition(UDim2.new(1, 50, 0.4, 0), "In", "Sine", 0.5)
end)

-- [[ 4. MAIN CONTROLLER ]] --
local function runSystem()
    -- เริ่มต้นด้วยการตรวจสอบบัคและเสนออัปเดตเวอร์ชันแรก
    task.spawn(function()
        task.wait(3)
        NotifDesc.Text = "Bug Fixed: Rapid Warp Engine\nReady for v" .. string.format("%.2f", VersionData.Current + 0.01)
        UpdateBox:TweenPosition(UDim2.new(1, -260, 0.4, 0), "Out", "Back", 0.5)
    end)

    -- ลูปวาร์ป 20 วินาที
    task.spawn(function()
        local countdown = 20
        while true do
            stableWarp()
            for i = 20, 1, -1 do
                countdown = i
                if isExpanded then
                    MainLabel.Text = "WINS: "..((player.leaderstats:FindFirstChild("Wins") or player.leaderstats:FindFirstChild("Win") or {Value=0}).Value).."\nWARP: "..countdown.."s"
                else
                    MainLabel.Text = "v" .. string.format("%.2f", VersionData.Current) .. " | " .. countdown .. "s"
                end
                task.wait(1)
            end
        end
    end)
end

-- [[ 5. INTERACTION ]] --
Island.MouseButton1Click:Connect(function()
    isExpanded = not isExpanded
    if isExpanded then
        Island:TweenSizeAndPosition(UDim2.new(0, 220, 0, 80), UDim2.new(0.5, -110, 0, 15), "Out", "Back", 0.3, true)
    else
        Island:TweenSizeAndPosition(UDim2.new(0, 150, 0, 32), UDim2.new(0.5, -75, 0, 15), "In", "Sine", 0.3, true)
    end
end)

-- หน้าใส่ Key
local function setupAuth()
    local KeyInput = Instance.new("TextBox")
    KeyInput.Parent = Island
    KeyInput.Size = UDim2.new(1, 0, 1, 0)
    KeyInput.BackgroundTransparency = 1
    KeyInput.Font = Enum.Font.GothamBold
    KeyInput.TextColor3 = Color3.new(1, 1, 1)
    KeyInput.PlaceholderText = "ENTER KEY"
    KeyInput.TextSize = 11
    
    KeyInput.FocusLost:Connect(function(enter)
        if enter and (KeyInput.Text == TARGET_KEY or KeyInput.Text == SPECIAL_KEY) then
            KeyInput:Destroy()
            runSystem()
        else
            KeyInput.Text = "" KeyInput.PlaceholderText = "WRONG KEY"
        end
    end)
end

setupAuth()

-- Anti-AFK
pcall(function()
    player.Idled:Connect(function()
        game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        task.wait(1)
        game:GetService("VirtualUser"):Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    end)
end)
