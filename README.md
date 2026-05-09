local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Library.CreateLib("AmNyamania 2 ✨ ANIMATED", "Midnight")

-- // Services
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- // Configuration
getgenv().candyEnabled = false
getgenv().autoFarm = false
getgenv().flyEnabled = false
getgenv().flySpeed = 60

-- // --- ANIMATION FUNCTIONS --- //

-- ฟังก์ชันวาร์ปแบบนุ่มนวล (Tween Animation)
local function SmoothTeleport(targetCFrame)
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return end
    local hrp = LocalPlayer.Character.HumanoidRootPart
    local distance = (hrp.Position - targetCFrame.Position).Magnitude
    local speed = 100 -- ความเร็วในการวาร์ป (ยิ่งเยอะยิ่งเร็ว)
    local duration = distance / speed

    local info = TweenInfo.new(duration, Enum.EasingStyle.Linear)
    local tween = TweenService:Create(hrp, info, {CFrame = targetCFrame})
    
    tween:Play()
    return tween
end

-- ระบบบินพร้อม Animation (Smooth Fly)
local function ToggleFly()
    local char = LocalPlayer.Character
    local hrp = char:WaitForChild("HumanoidRootPart")
    
    local bg = Instance.new("BodyGyro", hrp)
    bg.maxTorque = Vector3.new(9e9, 9e9, 9e9)
    bg.P = 5000 -- ปรับให้หมุนนุ่มนวล
    
    local bv = Instance.new("BodyVelocity", hrp)
    bv.maxForce = Vector3.new(9e9, 9e9, 9e9)
    
    task.spawn(function()
        while getgenv().flyEnabled do
            bg.cframe = workspace.CurrentCamera.CFrame
            bv.velocity = workspace.CurrentCamera.CFrame.LookVector * getgenv().flySpeed
            task.wait()
        end
        -- จบการบินแบบนุ่มนวล
        TweenService:Create(bg, TweenInfo.new(0.5), {P = 0}):Play()
        task.wait(0.5)
        bg:Destroy()
        bv:Destroy()
    end)
end

-- // --- UI TABS --- //

-- [TAB 1: AUTO FARM]
local Main = Window:NewTab("Auto Farm")
local CandySection = Main:NewSection("🍭 Lolipop System")

CandySection:NewToggle("Auto Candy (Smooth)", "เก็บลูกอมแบบมี Animation ป้องกันการโดนเด้ง", function(state)
    getgenv().candyEnabled = state
    if state then
        task.spawn(function()
            while getgenv().candyEnabled do
                for i = 1, 30 do
                    if not getgenv().candyEnabled then break end
                    pcall(function()
                        game:GetService("ReplicatedStorage").ReplicatedStorage_Source.Packages.Knit.Services.LolipopService.RF.CollectRegularLolipop:InvokeServer("Regular_" .. i)
                    end)
                    task.wait(0.05)
                end
                task.wait(1)
            end
        end)
    end
end)

local QuestSection = Main:NewSection("🏆 UGC Quests")
QuestSection:NewButton("Auto Collect All Om Nom", "วาร์ปเก็บไอเทมทุกอย่างแบบนุ่มนวล", function()
    for _, v in pairs(workspace:GetDescendants()) do
        if v.Name:lower():find("omnom") and (v:IsA("BasePart") or v:IsA("MeshPart")) then
            local tw = SmoothTeleport(v.CFrame)
            if tw then tw.Completed:Wait() end -- รอให้เดินไปถึงก่อนค่อยไปตัวต่อไป
            task.wait(0.3)
        end
    end
end)

-- [TAB 2: MOVEMENT]
local Move = Window:NewTab("Movement")
local FlySection = Move:NewSection("🚀 Advanced Fly")

FlySection:NewToggle("Fly Mode", "บินไปตามกล้อง (W,A,S,D)", function(state)
    getgenv().flyEnabled = state
    if state then ToggleFly() end
end)

FlySection:NewSlider("Fly Speed", "ความเร็วการร่อน", 300, 50, function(s)
    getgenv().flySpeed = s
end)

FlySection:NewButton("Infinite Jump", "กระโดดบนอากาศได้ไม่จำกัด", function()
    game:GetService("UserInputService").JumpRequest:Connect(function()
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
    end)
end)

-- [TAB 3: VISUALS & SETTINGS]
local Settings = Window:NewTab("Settings")
local VisualSection = Settings:NewSection("UI Customization")

VisualSection:NewKeybind("Toggle UI", "กดเพื่อซ่อนเมนู", Enum.KeyCode.RightControl, function()
    Library:ToggleUI()
end)

VisualSection:NewLabel("Animated Edition v2.0")
VisualSection:NewLabel("Credit: Gubby & Gemini")

-- Notify Success
Library:Notify("Script Loaded!", "ระบบฟาร์มพร้อมทำงานแล้ว", 5)
