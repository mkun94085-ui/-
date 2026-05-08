-- [[ R-HUB ULTIMATE | COMPLETE VERSION 1.0.0 ]] --

local TARGET_KEY = "six seven"
local SPECIAL_KEY = "Mark"
local EXPIRE_HOURS = 24
local FILE_NAME = "RH_Master_V1.txt"

local player = game:GetService("Players").LocalPlayer
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")

local currentVer = "1.0.0"
local isExpanded = false

-- [[ 1. STABLE REJOIN LOGIC ]] --
local function safeRejoin()
    pcall(function()
        -- ตรวจสอบความพร้อมก่อนรีจอย ป้องกันอาการรีจอยรัว
        MainLabel.Text = "⏳ PREPARING SAFE REJOIN..."
        task.wait(2)
        if #game:GetService("Players"):GetPlayers() <= 1 then
            -- ถ้าอยู่คนเดียว ให้รอห้องใหม่นานขึ้นป้องกันบัคหาห้องไม่เจอ
            task.wait(5)
        end
        TeleportService:Teleport(game.PlaceId, player)
    end)
end

-- [[ 2. UI CONSTRUCTION ]] --
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RHUB_V1_Master"
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
Instance.new("UICorner", Island).CornerRadius = UDim.new(1, 0)

local MainLabel = Instance.new("TextLabel")
MainLabel.Parent = Island
MainLabel.Size = UDim2.new(1, 0, 1, 0)
MainLabel.BackgroundTransparency = 1
MainLabel.Font = Enum.Font.GothamBold
MainLabel.TextColor3 = Color3.new(1, 1, 1)
MainLabel.TextSize = 11
MainLabel.Text = "R-HUB BOOTING..."

-- Update Notif Box
local UpdateBox = Instance.new("Frame")
UpdateBox.Parent = ScreenGui
UpdateBox.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
UpdateBox.Position = UDim2.new(1, 20, 0.4, 0)
UpdateBox.Size = UDim2.new(0, 220, 0, 80)
Instance.new("UICorner", UpdateBox)

local NotifTitle = Instance.new("TextLabel")
NotifTitle.Parent = UpdateBox
NotifTitle.Size = UDim2.new(1, 0, 0.4, 0)
NotifTitle.BackgroundTransparency = 1
NotifTitle.Font = Enum.Font.GothamBold
NotifTitle.TextColor3 = Color3.fromRGB(0, 255, 150)
NotifTitle.Text = "SYSTEM UPDATE"
NotifTitle.TextSize = 13

local NotifProgress = Instance.new("Frame")
NotifProgress.Parent = UpdateBox
NotifProgress.BackgroundColor3 = Color3.fromRGB(0, 255, 150)
NotifProgress.Position = UDim2.new(0.1, 0, 0.8, 0)
NotifProgress.Size = UDim2.new(0, 0, 0, 4)
Instance.new("UICorner", NotifProgress)

-- [[ 3. WARP ENGINE ]] --
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

local function executeWarp()
    pcall(function()
        local char = player.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local finishCF = getFinishCFrame()
        if root and finishCF then
            root.Velocity = Vector3.zero
            root.CFrame = finishCF * CFrame.new(0, 3, 0)
            root.Anchored = true; task.wait(0.4); root.Anchored = false
        end
    end)
end

-- [[ 4. THE FIRST LAUNCH (AUTO-UPDATE) ]] --
local function startUpdateSequence()
    Island:TweenSize(UDim2.new(0, 220, 0, 32), "Out", "Quart", 0.5, true)
    MainLabel.Text = "📡 SYNCING WITH CLOUD..."
    task.wait(1.5)
    
    -- เด้งแจ้งเตือนอัพเดตเวอร์ชันแรก
    UpdateBox:TweenPosition(UDim2.new(1, -230, 0.4, 0), "Out", "Back", 0.5)
    NotifTitle.Text = "INSTALLING V" .. currentVer
    NotifProgress:TweenSize(UDim2.new(0.8, 0, 0, 4), "Out", "Linear", 3)
    task.wait(3.5)
    
    UpdateBox:TweenPosition(UDim2.new(1, 20, 0.4, 0), "In", "Sine", 0.5)
    MainLabel.Text = "✅ VERSION " .. currentVer .. " INSTALLED"
    task.wait(1)
    
    -- เริ่มการทำงานจริง
    Island:TweenSize(UDim2.new(0, 150, 0, 32), "In", "Sine", 0.5, true)
    MainLabel.Text = "🕒 " .. os.date("%X")
    
    -- Loop ฟาร์ม (20s Warp | 5m Rejoin)
    local warpTimer = 20
    local rejoinTimer = 300
    
    task.spawn(function()
        while true do
            executeWarp()
            warpTimer = 20
            for i = 20, 1, -1 do
                warpTimer = i
                if isExpanded then
                    MainLabel.Text = "🏆 WINS: "..((player.leaderstats:FindFirstChild("Wins") or {Value=0}).Value).."\n🚀 NEXT WARP: "..warpTimer.."s"
                else
                    MainLabel.Text = "🕒 "..os.date("%X")
                end
                task.wait(1)
                rejoinTimer = rejoinTimer - 1
                
                if rejoinTimer <= 0 then safeRejoin() break end
            end
        end
    end)
end

-- [[ 5. INITIALIZE & AUTH ]] --
Island.MouseButton1Click:Connect(function()
    isExpanded = not isExpanded
    if isExpanded then
        Island:TweenSizeAndPosition(UDim2.new(0, 220, 0, 80), UDim2.new(0.5, -110, 0, 15), "Out", "Back", 0.3, true)
    else
        Island:TweenSizeAndPosition(UDim2.new(0, 150, 0, 32), UDim2.new(0.5, -75, 0, 15), "In", "Sine", 0.3, true)
    end
end)

-- หน้าใส่ Key (ถ้าไม่มีเซฟ)
local function authView()
    MainLabel.Text = ""
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
            startUpdateSequence()
        else
            KeyInput.Text = "" KeyInput.PlaceholderText = "WRONG KEY"
        end
    end)
end

authView()

-- Anti-AFK
pcall(function() player.Idled:Connect(function() game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame) end) end)
