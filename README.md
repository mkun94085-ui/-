-- [[ MARKW SYSTEM: ONE-CLICK FRN PASS MAX ]]
-- เป้าหมาย: เลเวล 100 ภายในปุ่มเดียว
-- ระบบ: Auto Decision + Multi-Remote Loop + Smart Build

repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local Root = LP.Character:WaitForChild("HumanoidRootPart")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

-- ระบบ Remote (ดึงจากสคริปต์ที่คุณใช้)
local Remotes = ReplicatedStorage:WaitForChild("RemoteEvents")
local CoinEvent = Remotes:WaitForChild("CollectIncome")
local GemEvent = Remotes:WaitForChild("RemoteCollectGem")
local SpinEvent = ReplicatedStorage:WaitForChild("RemoteFunctions"):WaitForChild("IncrementSpinAvailable")

_G.MasterSwitch = false

-- [[ UI DESIGN: CENTERED MODERN ]]
local Gui = Instance.new("ScreenGui", LP.PlayerGui)
local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.new(0, 260, 0, 180)
Main.Position = UDim2.new(0.5, -130, 0.5, -90)
Main.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
local Corner = Instance.new("UICorner", Main)
Corner.CornerRadius = UDim.new(0, 20)

local Stroke = Instance.new("UIStroke", Main)
Stroke.Thickness = 3
Stroke.Color = Color3.fromRGB(0, 255, 150)

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 50)
Title.Text = "MARKW FRN PASS MAX"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.BackgroundTransparency = 1

-- [[ ไม้ตาย: ฟังก์ชันกวาดล้างทุกอย่าง (Build & Collect) ]]
local function MasterLoop()
    pcall(function()
        -- 1. หา Tycoon
        local MyTycoon = nil
        for _,v in pairs(workspace:GetChildren()) do
            if v:IsA("Model") and v.Name:lower():find("tycoon") then
                local p = v:FindFirstChildWhichIsA("BasePart", true)
                if p and (Root.Position - p.Position).Magnitude < 150 then
                    MyTycoon = v; break
                end
            end
        end

        if MyTycoon then
            -- 2. สแกนหาปุ่มที่สร้างได้
            for _,v in pairs(MyTycoon:GetDescendants()) do
                if v:IsA("BasePart") and (v.Name:lower():find("button") or v.Name:lower():find("pad")) then
                    if v.Transparency < 0.5 then
                        -- วาร์ปไปเหยียบแบบเสี้ยววินาที
                        local oldCF = Root.CFrame
                        Root.CFrame = v.CFrame + Vector3.new(0, 3, 0)
                        firetouchinterest(Root, v, 0)
                        task.wait(0.05)
                        firetouchinterest(Root, v, 1)
                        Root.CFrame = oldCF
                    end
                end
            end
        end
    end)
end

-- [[ ปุ่มเดียวเสียวทั้งเซิร์ฟ ]]
local MasterBtn = Instance.new("TextButton", Main)
MasterBtn.Size = UDim2.new(0, 220, 0, 80)
MasterBtn.Position = UDim2.new(0.5, -110, 0, 70)
MasterBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MasterBtn.Text = "START AFK PASS 100%"
MasterBtn.TextColor3 = Color3.new(1, 1, 1)
MasterBtn.Font = Enum.Font.GothamBlack
MasterBtn.TextSize = 18
Instance.new("UICorner", MasterBtn).CornerRadius = UDim.new(0, 15)

MasterBtn.MouseButton1Click:Connect(function()
    _G.MasterSwitch = not _G.MasterSwitch
    MasterBtn.Text = _G.MasterSwitch and "SYSTEM ACTIVE" or "START AFK PASS 100%"
    MasterBtn.BackgroundColor3 = _G.MasterSwitch and Color3.fromRGB(0, 150, 100) or Color3.fromRGB(30, 30, 30)
end)

-- [[ HEARTBEAT: RUN EVERYTHING ]]
local t1, t2 = 0, 0
RunService.Heartbeat:Connect(function(dt)
    if _G.MasterSwitch then
        t1 += dt
        t2 += dt
        
        -- ปั๊มค่าเงินและเพชรรัวๆ (Bypass Remote)
        if t1 > 0.1 then
            t1 = 0
            pcall(function() CoinEvent:FireServer() end)
            pcall(function() GemEvent:FireServer() end)
            pcall(function() SpinEvent:InvokeServer(999999) end)
        end
        
        -- สั่งสร้างบ้าน
        if t2 > 1 then
            t2 = 0
            task.spawn(MasterLoop)
        end
        
        -- Rainbow Effect
        Stroke.Color = Color3.fromHSV((tick() * 0.2) % 1, 0.8, 1)
        Title.TextColor3 = Stroke.Color
    end
end)

-- Anti-AFK
LP.Idled:Connect(function()
    game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    task.wait(1)
    game:GetService("VirtualUser"):Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
end)

print("MARKW X FIER: ONE-CLICK MAX PASS READY!")
