--[[
    ╔══════════════════════════════════════════════╗
    ║            MARKW DYNAMIC SYSTEM              ║
    ║     ROLL | AUTO TOUCH | AUTO E-PROMPT        ║
    ╚══════════════════════════════════════════════╝
]]

local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")

local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()

-- // CONFIGURATION // --
local Config = {
    AutoRoll = false,
    AutoClaim = true,
    RollDelay = 0.6,
    ClaimRadius = 50, -- ระยะที่เหมาะสมเพื่อป้องกันการเตะจากระบบ Anti-Cheat
}

-- // UI DYNAMIC CONSTRUCTION // --
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MarkW_Dynamic_v2"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false

-- Main Frame (Dynamic Design)
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 350, 0, 450)
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -225) -- กึ่งกลางหน้าจอ
MainFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
MainFrame.BackgroundTransparency = 0.15
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 20)
UICorner.Parent = MainFrame

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Color3.fromRGB(80, 80, 80)
UIStroke.Thickness = 1.5
UIStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
UIStroke.Parent = MainFrame

-- Gradient Background
local UIGradient = Instance.new("UIGradient")
UIGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(20, 20, 20)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(40, 40, 40))
}
UIGradient.Rotation = 45
UIGradient.Parent = MainFrame

-- Header Title
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 60)
Title.BackgroundTransparency = 1
Title.Text = "MARKW DYNAMIC"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 24
Title.Font = Enum.Font.GothamBold
Title.Parent = MainFrame

-- Result Display (Glowing Effect)
local Display = Instance.new("Frame")
Display.Size = UDim2.new(0.85, 0, 0, 100)
Display.Position = UDim2.new(0.075, 0, 0.18, 0)
Display.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Display.BackgroundTransparency = 0.4
Display.Parent = MainFrame

local DisplayCorner = Instance.new("UICorner")
DisplayCorner.CornerRadius = UDim.new(0, 15)
DisplayCorner.Parent = Display

local ResultText = Instance.new("TextLabel")
ResultText.Size = UDim2.new(1, 0, 1, 0)
ResultText.BackgroundTransparency = 1
ResultText.Text = "STATUS: READY"
ResultText.TextColor3 = Color3.fromRGB(0, 255, 127)
ResultText.TextSize = 28
ResultText.Font = Enum.Font.GothamBold
ResultText.Parent = Display

-- // FUNCTIONAL LOGIC // --

local isRolling = false
local function rollEffect()
    if isRolling then return end
    isRolling = true
    
    -- อนิเมชั่นหน้าจอขณะ Roll
    TweenService:Create(ResultText, TweenInfo.new(0.1), {TextTransparency = 0.5}):Play()
    
    local items = {"COMMON", "RARE", "EPIC", "LEGENDARY", "MARKW SPECIAL"}
    for i = 1, 10 do
        ResultText.Text = items[math.random(1, #items)]
        ResultText.TextColor3 = Color3.fromHSV(tick() % 1, 0.8, 1)
        task.wait(0.05)
    end
    
    ResultText.TextTransparency = 0
    ResultText.Text = "COMPLETED"
    isRolling = false
end

-- // DYNAMIC BUTTON CREATOR // --
local function createDynamicBtn(text, pos, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.85, 0, 0, 55)
    btn.Position = pos
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(200, 200, 200)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.AutoButtonColor = false
    btn.Parent = MainFrame
    
    local bCorner = Instance.new("UICorner")
    bCorner.CornerRadius = UDim.new(0, 12)
    bCorner.Parent = btn

    -- Hover Animations
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(50, 50, 50), TextColor3 = Color3.white}):Play()
    end)
    btn.MouseLeave:Connect(function()
        if not string.find(btn.Text, "ON") then
            TweenService:Create(btn, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(30, 30, 30), TextColor3 = Color3.fromRGB(200, 200, 200)}):Play()
        end
    end)
    
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- Create Buttons
local RollBtn = createDynamicBtn("MANUAL ROLL", UDim2.new(0.075, 0, 0.45, 0), rollEffect)

local AutoRollBtn; AutoRollBtn = createDynamicBtn("AUTO ROLL: OFF", UDim2.new(0.075, 0, 0.6, 0), function()
    Config.AutoRoll = not Config.AutoRoll
    AutoRollBtn.Text = Config.AutoRoll and "AUTO ROLL: ON" or "AUTO ROLL: OFF"
    TweenService:Create(AutoRollBtn, TweenInfo.new(0.3), {BackgroundColor3 = Config.AutoRoll and Color3.fromRGB(100, 50, 200) or Color3.fromRGB(30, 30, 30)}):Play()
end)

local AutoClaimBtn; AutoClaimBtn = createDynamicBtn("AUTO COLLECT: ON", UDim2.new(0.075, 0, 0.75, 0), function()
    Config.AutoClaim = not Config.AutoClaim
    AutoClaimBtn.Text = Config.AutoClaim and "AUTO COLLECT: ON" or "AUTO COLLECT: OFF"
    TweenService:Create(AutoClaimBtn, TweenInfo.new(0.3), {BackgroundColor3 = Config.AutoClaim and Color3.fromRGB(0, 150, 100) or Color3.fromRGB(30, 30, 30)}):Play()
end)

-- // MASTER LOOP (AUTO ROLL & CLAIM) // --
task.spawn(function()
    while true do
        task.wait(0.1)
        local hrp = Character:FindFirstChild("HumanoidRootPart")
        if not hrp then continue end

        -- 1. Auto Roll Logic
        if Config.AutoRoll and not isRolling then
            task.spawn(rollEffect)
            task.wait(Config.RollDelay)
        end

        -- 2. Auto Claim Logic (Touch + E-Prompt)
        if Config.AutoClaim then
            for _, obj in pairs(workspace:GetDescendants()) do
                if obj:IsA("ProximityPrompt") then
                    -- ตรวจสอบ ProximityPrompt (ปุ่ม E)
                    local parent = obj.Parent
                    if parent and parent:IsA("BasePart") and (hrp.Position - parent.Position).Magnitude <= Config.ClaimRadius then
                        fireproximityprompt(obj)
                    end
                elseif obj:IsA("TouchTransmitter") then
                    -- ตรวจสอบการ Touch (เหรียญ/ของตกพื้น)
                    local parent = obj.Parent
                    if parent and parent:IsA("BasePart") and (hrp.Position - parent.Position).Magnitude <= Config.ClaimRadius then
                        firetouchinterest(hrp, parent, 0)
                        firetouchinterest(hrp, parent, 1)
                    end
                end
            end
        end
    end
end)

-- Drag System
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; dragStart = input.Position; startPos = MainFrame.Position end
end)
UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)

-- Intro Animation
MainFrame.Position = UDim2.new(0.5, -175, 1, 0)
TweenService:Create(MainFrame, TweenInfo.new(0.8, Enum.EasingStyle.Back), {Position = UDim2.new(0.5, -175, 0.5, -225)}):Play()

print("MARKW DYNAMIC SYSTEM LOADED SUCCESSFULLY")
