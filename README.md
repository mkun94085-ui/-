-- [[ FIER X MARKW: GHOST FLY SAFE EDITION ]]
-- แก้ไขระบบเก็บเงินไม่ทำงาน และเพิ่มความปลอดภัยสูงสุด

repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local LP = Players.LocalPlayer

repeat task.wait() until LP and LP:FindFirstChild("PlayerGui")

if getgenv().FIER_MARKW then
    getgenv().FIER_MARKW:Disconnect()
    getgenv().FIER_MARKW = nil
end

-- Remote Events
local Remotes = ReplicatedStorage:WaitForChild("RemoteEvents")
local CoinEvent = Remotes:WaitForChild("CollectIncome")
local GemEvent = Remotes:WaitForChild("RemoteCollectGem")
local SpinEvent = ReplicatedStorage:WaitForChild("RemoteFunctions"):WaitForChild("IncrementSpinAvailable")

-- ระบบกรองปุ่มเสียเงิน
local BAD = {"robux","upgrade","dev","product","premium","gamepass","donate"}
local function IsBad(obj)
    local cur = obj
    while cur do
        local n = cur.Name:lower()
        for _,b in ipairs(BAD) do if n:find(b) then return true end end
        cur = cur.Parent
    end
    return false
end

-- [[ ระบบเคลื่อนที่แบบลอย (Safe Stealth) ]]
local function GhostMove(targetPart)
    local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not hrp or not targetPart then return end
    
    local dist = (hrp.Position - targetPart.Position).Magnitude
    if dist > 150 then return end -- ป้องกันลอยออกนอกบ้าน

    hrp.Velocity = Vector3.new(0,0,0) -- ล้างความเร็วกันโดนเตะ
    local tween = TweenService:Create(hrp, TweenInfo.new(dist/20, Enum.EasingStyle.Linear), {
        CFrame = targetPart.CFrame + Vector3.new(0, 3, 0)
    })
    tween:Play()
    tween.Completed:Wait()
    
    firetouchinterest(hrp, targetPart, 0)
    task.wait(0.1)
    firetouchinterest(hrp, targetPart, 1)
end

-- [[ ค้นหา Tycoon และสิ่งของ ]]
local MyTycoon = nil
local Parts, Collectors = {}, {}

local function RefreshData()
    local char = LP.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    
    -- หา Tycoon ที่ใกล้ที่สุด
    for _,v in pairs(workspace:GetChildren()) do
        if v:IsA("Model") and v.Name:lower():find("tycoon") then
            local p = v:FindFirstChildWhichIsA("BasePart", true)
            if p and (char.HumanoidRootPart.Position - p.Position).Magnitude < 150 then
                MyTycoon = v
                break
            end
        end
    end

    if MyTycoon then
        local pT, cT = {}, {}
        for _,v in pairs(MyTycoon:GetDescendants()) do
            if v:IsA("BasePart") and not IsBad(v) then
                local n = v.Name:lower()
                -- ปรับปรุงการค้นหาปุ่มและที่เก็บเงิน
                if n:find("button") or n:find("pad") or n:find("buy") then table.insert(pT, v) end
                if n:find("collect") or n:find("deposit") or n:find("bank") then table.insert(cT, v) end
            end
        end
        Parts, Collectors = pT, cT
    end
end

task.spawn(function() while true do RefreshData() task.wait(2) end end)

-- [[ UI SYSTEM: CENTERED & ROUNDED ]]
local Gui = Instance.new("ScreenGui", LP.PlayerGui)
Gui.Name = "MARKW_ULTIMATE"

local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.new(0, 320, 0, 360)
Main.Position = UDim2.new(0.5, -160, 0.5, -180)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 15)
local Stroke = Instance.new("UIStroke", Main)
Stroke.Thickness = 3

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 50)
Title.Text = "MARKW X FIER PREMIUM"
Title.Font = Enum.Font.GothamBold
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundTransparency = 1

local Toggles = {IC=false, IG=false, AT=false, AC=false, IS=false}
local function AddToggle(txt, y, key)
    local B = Instance.new("TextButton", Main)
    B.Size = UDim2.new(0, 280, 0, 40)
    B.Position = UDim2.new(0.5, -140, 0, y)
    B.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    B.Text = txt.." : ปิด"
    B.TextColor3 = Color3.new(1, 1, 1)
    B.Font = Enum.Font.Gotham
    Instance.new("UICorner", B)
    B.MouseButton1Click:Connect(function()
        Toggles[key] = not Toggles[key]
        B.Text = txt.." : "..(Toggles[key] and "เปิด" or "ปิด")
        B.BackgroundColor3 = Toggles[key] and Color3.fromRGB(0, 180, 100) or Color3.fromRGB(30, 30, 30)
    end)
end

AddToggle("ปั๊มเหรียญ (Remote)", 60, "IC")
AddToggle("ปั๊มเพชร (Remote)", 110, "IG")
AddToggle("ออโต้สร้างบ้าน (ลอยไป)", 160, "AT")
AddToggle("ออโต้เก็บเงิน (ลอยไป)", 210, "AC")
AddToggle("ปั๊มสปิน", 260, "IS")

-- [[ MAIN LOOP ]]
local t1, t2, t3, t4, t5 = 0,0,0,0,0
local pi, ci = 1, 1

getgenv().FIER_MARKW = RunService.Heartbeat:Connect(function(dt)
    t1+=dt t2+=dt t3+=dt t4+=dt t5+=dt
    Stroke.Color = Color3.fromHSV((tick()*0.2)%1, 0.8, 1)
    Title.TextColor3 = Stroke.Color

    -- ปั๊ม Remote (ปลอดภัยขึ้น)
    if Toggles.IC and t1 > 0.5 then t1=0 pcall(function() CoinEvent:FireServer() end) end
    if Toggles.IG and t2 > 0.5 then t2=0 pcall(function() GemEvent:FireServer() end) end
    if Toggles.IS and t3 > 1.0 then t3=0 pcall(function() SpinEvent:InvokeServer(500) end) end

    -- ลอยไปสร้างบ้าน
    if Toggles.AT and t4 > 2.0 then
        t4=0
        if #Parts > 0 then
            local p = Parts[pi]
            if p and p.Transparency < 0.5 then GhostMove(p) end
            pi = (pi >= #Parts) and 1 or pi + 1
        end
    end

    -- ลอยไปเก็บเงินที่เครื่อง (แก้ไขแล้ว)
    if Toggles.AC and t5 > 2.5 then
        t5=0
        if #Collectors > 0 then
            local c = Collectors[ci]
            if c then GhostMove(c) end
            ci = (ci >= #Collectors) and 1 or ci + 1
        end
    end
end)

-- ปุ่มเปิด/ปิด
local Open = Instance.new("TextButton", Gui)
Open.Size = UDim2.new(0, 100, 0, 40)
Open.Position = UDim2.new(0, 10, 0.5, 0)
Open.Text = "เปิดเมนู"
Open.BackgroundColor3 = Color3.fromRGB(25,25,25)
Open.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", Open)
Open.MouseButton1Click:Connect(function() Main.Visible = not Main.Visible end)
