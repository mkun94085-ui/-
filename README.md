-- [[ R-HUB AETHER | v2.9.0 - UNIVERSAL WARP FIX ]] --

local TARGET_KEY = "six seven"
local ADMIN_KEY = "R-HUB-ADMIN-2026"
local SAVE_FILE = "RH_Aether_Final.json"

local player = game:GetService("Players").LocalPlayer
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")

local currentVer = 2.90
local isExpanded = false
local isWarping = false

-- [[ 1. UNIVERSAL WIN FINDER (เครื่องยนต์หาเส้นชัยใหม่) ]] --
local function getWinningCFrame()
    local target = nil
    -- ลำดับการหา: 1. ชื่อตรงตัว | 2. ชื่อคล้าย | 3. จุดที่มี TouchTransmitter (เส้นชัยส่วนใหญ่ใช้ตัวนี้)
    local winNames = {"finish", "win", "end", "goal", "victory", "checkpoint"}
    
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") then
            local name = v.Name:lower()
            for _, winName in pairs(winNames) do
                if name:find(winName) then
                    -- เช็คว่ามี TouchTransmitter ไหม (ถ้ามีคือจุดรับรางวัลชัวร์)
                    if v:FindFirstChildOfClass("TouchTransmitter") then
                        return v.CFrame
                    end
                    target = v.CFrame
                end
            end
        end
    end
    return target
end

-- [[ 2. CORE WARP LOGIC ]] --
local function executeWarp()
    if isWarping then return end
    isWarping = true
    
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local winCF = getWinningCFrame()
    
    if root and winCF then
        pcall(function()
            root.Velocity = Vector3.new(0,0,0)
            root.CFrame = winCF * CFrame.new(0, 3, 0)
            root.Anchored = true
            task.wait(0.8) -- ล็อกตัวกันเด้งกลับ
            root.Anchored = false
        end)
    else
        MainLabel.Text = "🔍 SEARCHING WIN..."
    end
    
    isWarping = false
end

-- [[ 3. UI & AUTH SYSTEM (24H) ]] --
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RH_Aether_Fix"
ScreenGui.Parent = player:WaitForChild("PlayerGui")
ScreenGui.IgnoreGuiInset = true

local Island = Instance.new("Frame")
Island.Parent = ScreenGui
Island.BackgroundColor3 = Color3.new(0, 0, 0)
Island.Position = UDim2.new(0.5, -65, 0, 15)
Island.Size = UDim2.new(0, 130, 0, 38)
Island.ClipsDescendants = true
Instance.new("UICorner", Island).CornerRadius = UDim.new(1, 0)

local UIStroke = Instance.new("UIStroke", Island)
UIStroke.Thickness = 1.5
UIStroke.Color = Color3.fromRGB(0, 255, 150)

local MainLabel = Instance.new("TextLabel")
MainLabel.Parent = Island
MainLabel.Size = UDim2.new(1, 0, 1, 0)
MainLabel.BackgroundTransparency = 1
MainLabel.Font = Enum.Font.GothamBold
MainLabel.TextColor3 = Color3.new(1, 1, 1)
MainLabel.TextSize = 10
MainLabel.Text = "AETHER v" .. string.format("%.2f", currentVer)

-- Update Panel
local UpdatePanel = Instance.new("Frame")
UpdatePanel.Parent = ScreenGui
UpdatePanel.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
UpdatePanel.Position = UDim2.new(1, 20, 0.4, 0)
UpdatePanel.Size = UDim2.new(0, 260, 0, 150)
Instance.new("UICorner", UpdatePanel)

local UpdateBtn = Instance.new("TextButton")
UpdateBtn.Parent = UpdatePanel
UpdateBtn.Position = UDim2.new(0.05, 0, 0.65, 0)
UpdateBtn.Size = UDim2.new(0, 235, 0, 38)
UpdateBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 150)
UpdateBtn.Font = Enum.Font.GothamBold
UpdateBtn.Text = "EVOLVE TO v" .. string.format("%.2f", currentVer + 0.1)
Instance.new("UICorner", UpdateBtn)

-- [[ 4. MAIN CONTROLLER ]] --
local function startFarming()
    task.spawn(function()
        while true do
            -- เช็ควันหมดอายุ (24 ชม.)
            if isfile(SAVE_FILE) then
                local data = HttpService:JSONDecode(readfile(SAVE_FILE))
                if (os.time() - data.startTime) >= 86400 then
                    MainLabel.Text = "🔒 EXPIRED"
                    break
                end
            end

            executeWarp()
            
            for i = 20, 1, -1 do
                if isExpanded then
                    MainLabel.Text = "WINS: "..((player.leaderstats:FindFirstChild("Wins") or {Value=0}).Value).."\nNEXT: "..i.."s"
                else
                    MainLabel.Text = i.."s | v"..string.format("%.2f", currentVer)
                end
                task.wait(1)
            end
        end
    end)
end

-- [[ 5. INTERACTION ]] --
Island.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        isExpanded = not isExpanded
        local ts = isExpanded and UDim2.new(0, 220, 0, 80) or UDim2.new(0, 130, 0, 38)
        TweenService:Create(Island, TweenInfo.new(0.4, Enum.EasingStyle.Back), {Size = ts}):Play()
    end
end)

UpdateBtn.MouseButton1Click:Connect(function()
    currentVer = currentVer + 0.1
    MainLabel.Text = "UPDATING..."
    task.wait(1.5)
    MainLabel.Text = "v" .. string.format("%.2f", currentVer)
    UpdatePanel:TweenPosition(UDim2.new(1, 20, 0.4, 0), "In", "Sine", 0.5)
    -- เปลี่ยนลูกเล่น UI หลังอัพเดต
    UIStroke.Color = Color3.fromHSV(tick()%1, 1, 1)
end)

-- Auth
local function init()
    local kIn = Instance.new("TextBox", Island)
    kIn.Size = UDim2.new(1,0,1,0); kIn.BackgroundTransparency = 1; kIn.TextColor3 = Color3.new(1,1,1); kIn.PlaceholderText = "24H KEY"
    kIn.FocusLost:Connect(function(enter)
        if enter then
            if kIn.Text == TARGET_KEY or kIn.Text == ADMIN_KEY then
                writefile(SAVE_FILE, HttpService:JSONEncode({startTime = os.time(), type = kIn.Text}))
                kIn:Destroy()
                startFarming()
                UpdatePanel:TweenPosition(UDim2.new(1, -280, 0.4, 0), "Out", "Back", 0.5)
            end
        end
    end)
end

init()
