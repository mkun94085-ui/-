-- [[ MARKW SYSTEM: ADMIN 045442 - FIXED UI ]]
-- ฟีเจอร์: มีช่องใส่ Code, ดูดทั้งแมพ, รีเบิร์ท, อัปเกรดฉลาด, ออโต้เคลม

repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Root = LP.Character:WaitForChild("HumanoidRootPart")

-- ตัวแปรระบบ
_G.IsAdmin = false
_G.MasterFarm = false
_G.AutoRebirth = false
_G.SmartUpgrade = false

-- [[ UI PREMIUM DESIGN ]]
local Gui = Instance.new("ScreenGui", LP.PlayerGui)
Gui.Name = "MARKW_ADMIN_FIXED"
Gui.ResetOnSpawn = false

local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.new(0, 340, 0, 450)
Main.Position = UDim2.new(0.5, -170, 0.5, -225)
Main.BackgroundColor3 = Color3.fromRGB(12, 12, 12)
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 20)

local Stroke = Instance.new("UIStroke", Main)
Stroke.Thickness = 3
Stroke.Color = Color3.fromRGB(255, 255, 255)

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 50)
Title.Text = "MARKW ADMIN HUB"
Title.Font = Enum.Font.GothamBlack
Title.TextSize = 18
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundTransparency = 1

-- [[ 1. ช่องใส่ CODE อัปเดต (เพิ่มให้แล้วครับ) ]]
local CodeBox = Instance.new("TextBox", Main)
CodeBox.Size = UDim2.new(0, 280, 0, 45)
CodeBox.Position = UDim2.new(0.5, -140, 0, 60)
CodeBox.PlaceholderText = "กรอก CODE (045442)..."
CodeBox.Text = ""
CodeBox.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
CodeBox.TextColor3 = Color3.new(1, 1, 1)
CodeBox.Font = Enum.Font.GothamBold
Instance.new("UICorner", CodeBox)

CodeBox.FocusLost:Connect(function(enter)
    if enter then
        if CodeBox.Text == "045442" then
            _G.IsAdmin = true
            CodeBox.Text = "CODE SUCCESS: ADMIN ACTIVE"
            CodeBox.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
            Title.Text = "MARKW ADMIN: 045442"
            Stroke.Color = Color3.fromRGB(0, 255, 200)
        else
            CodeBox.Text = "CODE INVALID!"
            task.wait(1)
            CodeBox.Text = ""
        end
    end
end)

-- [[ 2. ระบบ MAGNET (ถั่ว & เพชร) ]]
task.spawn(function()
    while task.wait(0.1) do
        if _G.MasterFarm then
            pcall(function()
                -- ถ้าใส่ Code แล้ว ดูดทั้งแมพ (99999) ถ้ายังไม่ใส่ ดูดระยะ 150
                local Range = _G.IsAdmin and 99999 or 150
                for _, v in pairs(workspace:GetDescendants()) do
                    if v:IsA("BasePart") and v.Transparency < 0.5 then
                        local n = v.Name:lower()
                        if n:find("nut") or n:find("gem") or n:find("token") or n:find("diamond") then
                            v.CFrame = Root.CFrame - Vector3.new(0, 3, 0)
                        end
                    end
                end
            end)
        end
    end
end)

-- [[ 3. ระบบ REBIRTH & UPGRADE ]]
task.spawn(function()
    while task.wait(1.5) do
        if _G.AutoRebirth then
            for _, r in pairs(ReplicatedStorage:GetDescendants()) do
                if r:IsA("RemoteEvent") and r.Name:lower():find("rebirth") then r:FireServer() end
            end
        end
        if _G.SmartUpgrade then
            for _, r in pairs(ReplicatedStorage:GetDescendants()) do
                if r:IsA("RemoteEvent") and (r.Name:lower():find("upgrade") or r.Name:lower():find("buy")) then
                    if r.Name:lower():find("multi") then for i=1,5 do r:FireServer(i) end end
                    r:FireServer(1)
                end
            end
        end
    end
end)

-- [[ 4. ระบบ AUTO CLAIM UGC ]]
task.spawn(function()
    while task.wait(10) do
        if _G.IsAdmin and _G.MasterFarm then
            pcall(function()
                if LP.leaderstats.Nuts.Value >= 1000000 then
                    local Claim = ReplicatedStorage:FindFirstChild("ClaimUGC", true) or ReplicatedStorage:FindFirstChild("Events"):FindFirstChild("Claim")
                    if Claim then Claim:FireServer() end
                end
            end)
        end
    end
end)

-- [[ ฟังก์ชันสร้างปุ่ม ]]
local function AddToggle(txt, y, key)
    local B = Instance.new("TextButton", Main)
    B.Size = UDim2.new(0, 280, 0, 50)
    B.Position = UDim2.new(0.5, -140, 0, y)
    B.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    B.Text = txt .. " : ปิด"
    B.TextColor3 = Color3.new(1, 1, 1)
    B.Font = Enum.Font.GothamBold
    Instance.new("UICorner", B).CornerRadius = UDim.new(0, 12)
    
    B.MouseButton1Click:Connect(function()
        _G[key] = not _G[key]
        B.Text = txt .. " : " .. (_G[key] and "เปิด" or "ปิด")
        B.BackgroundColor3 = _G[key] and Color3.fromRGB(0, 180, 100) or Color3.fromRGB(30, 30, 30)
    end)
end

AddToggle("เริ่มดูดของอัตโนมัติ", 120, "MasterFarm")
AddToggle("อัปเกรดอัตโนมัติ", 185, "SmartUpgrade")
AddToggle("รีเบิร์ทอัตโนมัติ", 250, "AutoRebirth")

-- ปุ่มเปิด/ปิดหน้าเมนู
local Tgl = Instance.new("TextButton", Gui)
Tgl.Size = UDim2.new(0, 80, 0, 35)
Tgl.Position = UDim2.new(0.5, -40, 0, 10)
Tgl.Text = "MENU"
Tgl.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Tgl.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", Tgl)
Tgl.MouseButton1Click:Connect(function() Main.Visible = not Main.Visible end)

-- RGB Effect & Anti-AFK
RunService.RenderStepped:Connect(function()
    local c = Color3.fromHSV(tick() % 5 / 5, 0.8, 1)
    if not _G.IsAdmin then Stroke.Color = c end
    Title.TextColor3 = c
end)
LP.Idled:Connect(function() game:GetService("VirtualUser"):ClickButton2(Vector2.new(0,0)) end)
