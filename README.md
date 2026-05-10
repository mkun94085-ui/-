--[[
    ╔══════════════════════════════════════════════╗
    ║                MARKW SYSTEM                  ║
    ║        ALL-IN-ONE: ROLL & AUTO CLAIM         ║
    ╚══════════════════════════════════════════════╝
]]

local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")

local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()

-- // CONFIGURATION // --
local Config = {
    AutoRoll = false,
    AutoClaim = true,
    RollDelay = 0.5,
    ClaimRadius = 500,
    ItemKey = "UGC"
}

-- // UI CONSTRUCTION // --
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MarkW_Ultimate_System"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 380, 0, 480)
MainFrame.Position = UDim2.new(0.5, -190, 0.5, -240) -- อยู่กลางหน้าจอเสมอ
MainFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 12)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 18)
UICorner.Parent = MainFrame

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Color3.fromRGB(35, 35, 35)
UIStroke.Thickness = 2
UIStroke.Parent = MainFrame

-- Header
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 60)
Title.BackgroundTransparency = 1
Title.Text = "MARKW SYSTEM"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 22
Title.Font = Enum.Font.GothamBold
Title.Parent = MainFrame

-- Display Result
local Display = Instance.new("Frame")
Display.Size = UDim2.new(0.88, 0, 0, 120)
Display.Position = UDim2.new(0.06, 0, 0.15, 0)
Display.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Display.Parent = MainFrame

local DisplayCorner = Instance.new("UICorner")
DisplayCorner.CornerRadius = UDim.new(0, 12)
DisplayCorner.Parent = Display

local ResultText = Instance.new("TextLabel")
ResultText.Size = UDim2.new(1, 0, 1, 0)
ResultText.BackgroundTransparency = 1
ResultText.Text = "READY"
ResultText.TextColor3 = Color3.fromRGB(0, 255, 150)
ResultText.TextSize = 35
ResultText.Font = Enum.Font.GothamBold
ResultText.Parent = Display

-- // FUNCTIONS // --

local isRolling = false
local function rollEffect()
    if isRolling then return end
    isRolling = true
    
    local items = {"COMMON", "RARE", "EPIC", "LEGENDARY", "MYTHIC"}
    local colors = {
        Color3.fromRGB(180, 180, 180),
        Color3.fromRGB(80, 255, 120),
        Color3.fromRGB(150, 100, 255),
        Color3.fromRGB(255, 200, 50),
        Color3.fromRGB(255, 50, 50)
    }

    for i = 1, 12 do
        local rand = math.random(1, #items)
        ResultText.Text = items[rand]
        ResultText.TextColor3 = colors[rand]
        task.wait(0.06)
    end
    
    -- จำลองโอกาสได้ UGC (ปรับตามระบบจริงของเกม)
    if math.random(1, 100) > 98 then
        ResultText.Text = "UGC UNLOCKED!"
        ResultText.TextColor3 = Color3.fromRGB(255, 255, 255)
    else
        ResultText.Text = "LUCK: " .. math.random(1, 999)
    end
    
    isRolling = false
end

-- // BUTTON CREATOR // --
local function createButton(name, pos, text, color)
    local btn = Instance.new("TextButton")
    btn.Name = name
    btn.Size = UDim2.new(0.88, 0, 0, 50)
    btn.Position = pos
    btn.BackgroundColor3 = color
    btn.Text = text
    btn.TextColor3 = Color3.white
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.AutoButtonColor = false
    btn.Parent = MainFrame
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = btn
    
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundTransparency = 0.2}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play()
    end)
    
    return btn
end

local RollBtn = createButton("RollBtn", UDim2.new(0.06, 0, 0.45, 0), "MANUAL ROLL", Color3.fromRGB(0, 120, 215))
local AutoRollBtn = createButton("AutoRollBtn", UDim2.new(0.06, 0, 0.58, 0), "AUTO ROLL: OFF", Color3.fromRGB(40, 40, 40))
local AutoClaimBtn = createButton("AutoClaimBtn", UDim2.new(0.06, 0, 0.71, 0), "AUTO CLAIM: ON", Color3.fromRGB(0, 170, 127))

-- // LOGIC HANDLERS // --

RollBtn.MouseButton1Click:Connect(rollEffect)

AutoRollBtn.MouseButton1Click:Connect(function()
    Config.AutoRoll = not Config.AutoRoll
    if Config.AutoRoll then
        AutoRollBtn.Text = "AUTO ROLL: ON"
        AutoRollBtn.BackgroundColor3 = Color3.fromRGB(170, 85, 255)
    else
        AutoRollBtn.Text = "AUTO ROLL: OFF"
        AutoRollBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    end
end)

AutoClaimBtn.MouseButton1Click:Connect(function()
    Config.AutoClaim = not Config.AutoClaim
    if Config.AutoClaim then
        AutoClaimBtn.Text = "AUTO CLAIM: ON"
        AutoClaimBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 127)
    else
        AutoClaimBtn.Text = "AUTO CLAIM: OFF"
        AutoClaimBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    end
end)

-- Loop: Auto Roll
task.spawn(function()
    while task.wait(Config.RollDelay) do
        if Config.AutoRoll and not isRolling then
            pcall(rollEffect)
        end
    end
end)

-- Loop: Auto Claim Ground Items
task.spawn(function()
    while task.wait(0.5) do
        if Config.AutoClaim then
            pcall(function()
                local hrp = Character:FindFirstChild("HumanoidRootPart")
                if not hrp then return end
                
                for _, obj in pairs(workspace:GetDescendants()) do
                    if obj:IsA("TouchTransmitter") and obj.Parent:IsA("Part") then
                        local item = obj.Parent
                        if (hrp.Position - item.Position).Magnitude <= Config.ClaimRadius then
                            firetouchinterest(hrp, item, 0)
                            task.wait()
                            firetouchinterest(hrp, item, 1)
                        end
                    end
                end
            end)
        end
    end
end)

-- Draggable UI
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; dragStart = input.Position; startPos = MainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

print("MARKW SYSTEM LOADED - UI CENTERED & READY")
