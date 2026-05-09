-- [[ R-HUB AETHER | ADVANCED UPDATE UI SYSTEM v2.8.0 ]] --

local TARGET_KEY = "six seven"
local ADMIN_KEY = "R-HUB-ADMIN-2026"
local SAVE_FILE = "RH_Aether_Cloud.json"

local player = game:GetService("Players").LocalPlayer
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")

local currentVer = 2.80
local isExpanded = false
local isWarping = false

-- [[ 1. STORAGE & EXPIRY ]] --
local function saveSession(kType)
    local data = {startTime = os.time(), kType = kType, ver = currentVer}
    writefile(SAVE_FILE, HttpService:JSONEncode(data))
end

local function getSession()
    if not isfile(SAVE_FILE) then return nil end
    return HttpService:JSONDecode(readfile(SAVE_FILE))
end

local function isExpired()
    local session = getSession()
    if not session then return true end
    return (os.time() - session.startTime) >= 86400 -- 24 ชม.
end

-- [[ 2. UI CREATION ]] --
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RH_Aether_Evolution"
ScreenGui.Parent = player:WaitForChild("PlayerGui")
ScreenGui.IgnoreGuiInset = true

-- Dynamic Island (หลัก)
local Island = Instance.new("Frame")
Island.Name = "Island"
Island.Parent = ScreenGui
Island.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Island.Position = UDim2.new(0.5, -65, 0, 15)
Island.Size = UDim2.new(0, 130, 0, 38)
Island.ClipsDescendants = true
local IslandCorner = Instance.new("UICorner", Island)
IslandCorner.CornerRadius = UDim.new(1, 0)

local UIStroke = Instance.new("UIStroke", Island)
UIStroke.Thickness = 1.5
UIStroke.Color = Color3.fromRGB(0, 255, 180)
UIStroke.Transparency = 0.4

local MainLabel = Instance.new("TextLabel")
MainLabel.Parent = Island
MainLabel.Size = UDim2.new(1, 0, 1, 0)
MainLabel.BackgroundTransparency = 1
MainLabel.Font = Enum.Font.GothamBold
MainLabel.TextColor3 = Color3.new(1, 1, 1)
MainLabel.TextSize = 11
MainLabel.Text = "AETHER v" .. string.format("%.2f", currentVer)

-- Update Center (หน้าต่างอัปเดต)
local UpdatePanel = Instance.new("Frame")
UpdatePanel.Parent = ScreenGui
UpdatePanel.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
UpdatePanel.Position = UDim2.new(1, 20, 0.4, 0) -- ซ่อนไว้ขวาจอ
UpdatePanel.Size = UDim2.new(0, 260, 0, 150)
Instance.new("UICorner", UpdatePanel)

local PanelStroke = Instance.new("UIStroke", UpdatePanel)
PanelStroke.Color = Color3.fromRGB(40, 40, 40)
PanelStroke.Thickness = 2

local Title = Instance.new("TextLabel")
Title.Parent = UpdatePanel
Title.Size = UDim2.new(1, 0, 0.3, 0)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamBold
Title.TextColor3 = Color3.fromRGB(0, 180, 255)
Title.TextSize = 14
Title.Text = "SYSTEM EVOLUTION"

local Desc = Instance.new("TextLabel")
Desc.Parent = UpdatePanel
Desc.Position = UDim2.new(0, 15, 0.3, 0)
Desc.Size = UDim2.new(1, -30, 0.3, 0)
Desc.BackgroundTransparency = 1
Desc.Font = Enum.Font.Gotham
Desc.TextColor3 = Color3.fromRGB(200, 200, 200)
Desc.TextSize = 10
Desc.Text = "New Patch v"..string.format("%.2f", currentVer + 0.1).." Ready.\n- Fixed UI Glitch\n- Boosted Warp Speed"
Desc.TextXAlignment = Enum.TextXAlignment.Left

local UpdateBtn = Instance.new("TextButton")
UpdateBtn.Parent = UpdatePanel
UpdateBtn.Position = UDim2.new(0.05, 0, 0.65, 0)
UpdateBtn.Size = UDim2.new(0, 235, 0, 38)
UpdateBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 150)
UpdateBtn.Font = Enum.Font.GothamBold
UpdateBtn.TextColor3 = Color3.new(0, 0, 0)
UpdateBtn.TextSize = 12
UpdateBtn.Text = "INSTALL UPDATE"
Instance.new("UICorner", UpdateBtn)

-- [[ 3. EVOLUTION LOGIC ]] --
local function applyEvolution()
    if currentVer >= 10.01 then return end
    
    currentVer = currentVer + 0.5 -- เพิ่มเวอร์ชัน
    
    -- เอฟเฟกต์การอัปเกรด UI
    UpdateBtn.Visible = false
    Title.Text = "INSTALLING..."
    task.wait(2)
    
    -- ปรับหน้าตา Island ตามเวอร์ชัน
    if currentVer >= 4.0 then
        UIStroke.Color = Color3.fromRGB(255, 0, 100) -- เปลี่ยนสีขอบ
        UIStroke.Thickness = 2.5
    end
    if currentVer >= 7.0 then
        Island.BackgroundColor3 = Color3.fromRGB(10, 0, 20)
        UIStroke.Color = Color3.fromRGB(255, 215, 0) -- ขอบทอง
    end
    
    MainLabel.Text = "v" .. string.format("%.2f", currentVer)
    UpdatePanel:TweenPosition(UDim2.new(1, 20, 0.4, 0), "In", "Sine", 0.5)
    UpdateBtn.Visible = true
    Title.Text = "SYSTEM EVOLUTION"
    Desc.Text = "System is up to date."
end

-- [[ 4. CORE LOOP ]] --
local function runAether()
    task.spawn(function()
        while true do
            if isExpired() then
                MainLabel.Text = "EXPIRED (24H)"
                MainLabel.TextColor3 = Color3.new(1,0,0)
                UIStroke.Color = Color3.new(1,0,0)
                UpdateBtn.Text = "KEY EXPIRED - CONTACT ADMIN"
                UpdateBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
                break
            end
            
            -- Stable Warp
            if not isWarping then
                isWarping = true
                pcall(function()
                    local root = player.Character.HumanoidRootPart
                    local win = workspace:FindFirstChild("Finish") or workspace:FindFirstChild("Win")
                    if root and win then
                        root.CFrame = win.CFrame * CFrame.new(0, 3, 0)
                        root.Anchored = true; task.wait(0.6); root.Anchored = false
                    end
                end)
                isWarping = false
            end
            
            -- Island Animation
            for i = 20, 1, -1 do
                if isExpanded then
                    MainLabel.Text = "WARP: "..i.."s | v"..string.format("%.2f", currentVer)
                else
                    MainLabel.Text = i .. "s | v" .. string.format("%.2f", currentVer)
                end
                task.wait(1)
            end
        end
    end)
    
    -- ระบบเด้งอัปเดตอัจฉริยะ (สุ่มเด้งทุก 5 ชม.)
    task.spawn(function()
        while true do
            task.wait(18000)
            if not isExpired() then
                UpdatePanel:TweenPosition(UDim2.new(1, -280, 0.4, 0), "Out", "Back", 0.5)
            end
        end
    end)
end

-- [[ 5. INITIALIZE ]] --
UpdateBtn.MouseButton1Click:Connect(function()
    if isExpired() then return end
    applyEvolution()
end)

Island.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        isExpanded = not isExpanded
        local ts = isExpanded and UDim2.new(0, 240, 0, 85) or UDim2.new(0, 130, 0, 38)
        local tp = isExpanded and UDim2.new(0.5, -120, 0, 15) or UDim2.new(0.5, -65, 0, 15)
        TweenService:Create(Island, TweenInfo.new(0.4, Enum.EasingStyle.Back), {Size = ts, Position = tp}):Play()
    end
end)

local function startAuth()
    local session = getSession()
    if session and not isExpired() then
        runAether()
    else
        MainLabel.Text = "NEED KEY"
        local kIn = Instance.new("TextBox", Island)
        kIn.Size = UDim2.new(1,0,1,0); kIn.BackgroundTransparency = 1; kIn.TextColor3 = Color3.new(1,1,1); kIn.PlaceholderText = "24H KEY"
        kIn.FocusLost:Connect(function(enter)
            if enter then
                if kIn.Text == TARGET_KEY or kIn.Text == ADMIN_KEY then
                    saveSession(kIn.Text == ADMIN_KEY and "ADMIN" or "USER")
                    kIn:Destroy(); runAether()
                    UpdatePanel:TweenPosition(UDim2.new(1, -280, 0.4, 0), "Out", "Back", 0.5) -- เด้ง UI อัปเดตทันทีที่รัน
                end
            end
        end)
    end
end

startAuth()
