-- [[ MARKW SYSTEM: ADMIN 045442 SPECIAL EDITION ]]
-- ฟีเจอร์: ดูดทั้งแมพ + รีเบิร์ทไว + อัปเกรดฉลาด + ออโต้เคลม UGC

repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Root = LP.Character:WaitForChild("HumanoidRootPart")

-- ค่าเริ่มต้นระบบ (ปลดล็อกด้วย Code แอดมินแล้ว)
_G.IsAdmin = true
_G.MasterFarm = false
_G.AutoRebirth = false
_G.SmartUpgrade = false

-- [[ UI PREMIUM DESIGN ]]
local Gui = Instance.new("ScreenGui", LP.PlayerGui)
Gui.Name = "MARKW_ADMIN_HUB"
Gui.ResetOnSpawn = false

local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.new(0, 340, 0, 420)
Main.Position = UDim2.new(0.5, -170, 0.5, -210)
Main.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 20)

local Stroke = Instance.new("UIStroke", Main)
Stroke.Thickness = 3
Stroke.Color = Color3.fromRGB(0, 255, 200)

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 60)
Title.Text = "MARKW ADMIN: 045442"
Title.Font = Enum.Font.GothamBlack
Title.TextSize = 18
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundTransparency = 1

-- [[ 1. ระบบ GLOBAL MAGNET (GOD MODE) ]]
task.spawn(function()
    while task.wait(0.05) do
        if _G.MasterFarm then
            pcall(function()
                -- รัศมี Infinity สำหรับแอดมิน
                for _, v in pairs(workspace:GetDescendants()) do
                    if v:IsA("BasePart") and v.Transparency < 0.5 then
                        local n = v.Name:lower()
                        if n:find("nut") or n:find("gem") or n:find("token") or n:find("diamond") then
                            -- ย้ายมาที่เท้าเราทันที
                            v.CFrame = Root.CFrame - Vector3.new(0, 3, 0)
                        end
                    end
                end
            end)
        end
    end
end)

-- [[ 2. ระบบ AUTO REBIRTH (FAST) ]]
task.spawn(function()
    while task.wait(1) do
        if _G.AutoRebirth then
            pcall(function()
                for _, r in pairs(ReplicatedStorage:GetDescendants()) do
                    if r:IsA("RemoteEvent") and r.Name:lower():find("rebirth") then
                        r:FireServer()
                    end
                end
            end)
        end
    end
end)

-- [[ 3. ระบบ SMART UPGRADE (PRIORITY) ]]
task.spawn(function()
    while task.wait(1.5) do
        if _G.SmartUpgrade then
            pcall(function()
                for _, r in pairs(ReplicatedStorage:GetDescendants()) do
                    if r:IsA("RemoteEvent") and (r.Name:lower():find("upgrade") or r.Name:lower():find("buy")) then
                        local rName = r.Name:lower()
                        -- เน้น Multiplier ก่อนเพื่อความรวย
                        if rName:find("multi") or rName:find("capa") then
                            for i=1,10 do r:FireServer(i) end
                        end
                        r:FireServer(1)
                    end
                end
            end)
        end
    end
end)

-- [[ 4. ระบบ AUTO CLAIM UGC TRACKER ]]
task.spawn(function()
    while task.wait(10) do
        pcall(function()
            local Nuts = LP.leaderstats.Nuts.Value
            if Nuts >= 1000000 then
                local ClaimEvent = ReplicatedStorage:FindFirstChild("ClaimUGC", true) or ReplicatedStorage:FindFirstChild("Events"):FindFirstChild("Claim")
                if ClaimEvent then 
                    ClaimEvent:FireServer() 
                    print("MARKW: UGC CLAIMED!")
                end
            end
        end)
    end
end)

-- [[ ฟังก์ชันปุ่มเมนู ]]
local function AddToggle(txt, y, key)
    local B = Instance.new("TextButton", Main)
    B.Size = UDim2.new(0, 280, 0, 55)
    B.Position = UDim2.new(0.5, -140, 0, y)
    B.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    B.Text = txt .. " : ปิด"
    B.TextColor3 = Color3.new(1, 1, 1)
    B.Font = Enum.Font.GothamBold
    Instance.new("UICorner", B).CornerRadius = UDim.new(0, 15)
    
    B.MouseButton1Click:Connect(function()
        _G[key] = not _G[key]
        B.Text = txt .. " : " .. (_G[key] and "เปิด" or "ปิด")
        B.BackgroundColor3 = _G[key] and Color3.fromRGB(0, 150, 100) or Color3.fromRGB(30, 30, 30)
    end)
end

AddToggle("ดูดถั่วและเพชรทั้งแมพ", 80, "MasterFarm")
AddToggle("อัปเกรดอัตโนมัติ (ฉลาด)", 150, "SmartUpgrade")
AddToggle("รีเบิร์ทอัตโนมัติ (ไว)", 220, "AutoRebirth")

-- ปุ่มแสดงสถานะ Admin
local Status = Instance.new("TextLabel", Main)
Status.Size = UDim2.new(0, 280, 0, 40)
Status.Position = UDim2.new(0.5, -140, 0, 300)
Status.Text = "ADMIN PRIVILEGE: 045442 ACTIVE"
Status.TextColor3 = Color3.fromRGB(255, 200, 0)
Status.Font = Enum.Font.GothamBold
Status.BackgroundTransparency = 1

-- ปุ่มเปิด/ปิดเมนู
local Tgl = Instance.new("TextButton", Gui)
Tgl.Size = UDim2.new(0, 80, 0, 35)
Tgl.Position = UDim2.new(0.5, -40, 0, 15)
Tgl.Text = "MENU"
Tgl.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Tgl.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", Tgl)
Tgl.MouseButton1Click:Connect(function() Main.Visible = not Main.Visible end)

-- Anti-AFK & RGB Effect
RunService.RenderStepped:Connect(function()
    local c = Color3.fromHSV(tick() % 5 / 5, 0.8, 1)
    Stroke.Color = c
    Title.TextColor3 = c
end)
LP.Idled:Connect(function() game:GetService("VirtualUser"):ClickButton2(Vector2.new(0,0)) end)
