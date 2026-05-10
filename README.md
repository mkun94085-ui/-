-- [[ MARKW SYSTEM: NUT COLLECTOR PREMIUM V.2 ]]
-- ฟีเจอร์: ดูดถั่วเข้าตัว (Magnet), อัปเกรดอัตโนมัติ, ระบบป้องกันการค้าง

repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Character = LP.Character or LP.CharacterAdded:Wait()
local Root = Character:WaitForChild("HumanoidRootPart")

_G.NutMagnet = false
_G.AutoUpgrade = false

-- [[ สร้าง UI สไตล์ MARKW ]]
local Gui = Instance.new("ScreenGui", LP.PlayerGui)
Gui.Name = "MARKW_PREMIUM_HUB"
Gui.ResetOnSpawn = false

local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.new(0, 320, 0, 240)
Main.Position = UDim2.new(0.5, -160, 0.5, -120)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Main.BorderSizePixel = 0
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 20)

-- ขอบ RGB แบบเนียนๆ
local Stroke = Instance.new("UIStroke", Main)
Stroke.Thickness = 3
Stroke.Color = Color3.fromRGB(255, 255, 255)

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 50)
Title.Text = "MARKW NUT COLLECTOR"
Title.Font = Enum.Font.GothamBlack
Title.TextSize = 18
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundTransparency = 1

-- [[ ฟังก์ชัน: ระบบดูดถั่ว (Magnet) ]]
task.spawn(function()
    while task.wait(0.1) do
        if _G.NutMagnet then
            pcall(function()
                -- สแกนหาถั่วในรัศมีรอบตัว
                for _, v in pairs(workspace:GetDescendants()) do
                    if v:IsA("BasePart") and v.Transparency < 0.5 then
                        local name = v.Name:lower()
                        if name:find("nut") or name:find("token") or name:find("collect") then
                            if (v.Position - Root.Position).Magnitude < 100 then
                                -- ดูดมาที่ตัวเราทันที
                                v.CFrame = Root.CFrame
                            end
                        end
                    end
                end
            end)
        end
    end
end)

-- [[ ฟังก์ชัน: ระบบอัปเกรดอัตโนมัติ ]]
task.spawn(function()
    while task.wait(1) do
        if _G.AutoUpgrade then
            pcall(function()
                -- สแกนหา Remote อัปเกรดในเครื่อง
                for _, r in pairs(ReplicatedStorage:GetDescendants()) do
                    if r:IsA("RemoteEvent") and (r.Name:lower():find("upgrade") or r.Name:lower():find("buy")) then
                        -- พยายามอัปเกรด Index 1 ถึง 10
                        for i = 1, 10 do
                            r:FireServer(i)
                        end
                    end
                end
            end)
        end
    end
end)

-- [[ ปุ่มเปิด/ปิด (Toggles) ]]
local function CreateToggle(txt, y, key)
    local B = Instance.new("TextButton", Main)
    B.Size = UDim2.new(0, 280, 0, 50)
    B.Position = UDim2.new(0.5, -140, 0, y)
    B.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    B.Text = txt .. " : ปิด"
    B.TextColor3 = Color3.new(1, 1, 1)
    B.Font = Enum.Font.GothamBold
    B.TextSize = 14
    Instance.new("UICorner", B).CornerRadius = UDim.new(0, 12)
    
    B.MouseButton1Click:Connect(function()
        _G[key] = not _G[key]
        B.Text = txt .. " : " .. (_G[key] and "เปิด" or "ปิด")
        B.BackgroundColor3 = _G[key] and Color3.fromRGB(0, 170, 100) or Color3.fromRGB(30, 30, 30)
        
        -- Animation เล็กน้อยตอนกด
        B:TweenSize(UDim2.new(0, 270, 0, 45), "Out", "Quad", 0.1, true)
        task.wait(0.1)
        B:TweenSize(UDim2.new(0, 280, 0, 50), "Out", "Quad", 0.1, true)
    end)
end

CreateToggle("ระบบดูดถั่วอัตโนมัติ", 70, "NutMagnet")
CreateToggle("ระบบอัปเกรดอัตโนมัติ", 130, "AutoUpgrade")

-- ปุ่มย่อ/ขยายเมนู (เปิด/ปิด)
local ToggleBtn = Instance.new("TextButton", Gui)
ToggleBtn.Size = UDim2.new(0, 100, 0, 40)
ToggleBtn.Position = UDim2.new(0.5, -50, 0, 15)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
ToggleBtn.Text = "เปิด/ปิด เมนู"
ToggleBtn.TextColor3 = Color3.new(1, 1, 1)
ToggleBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", ToggleBtn)

ToggleBtn.MouseButton1Click:Connect(function()
    Main.Visible = not Main.Visible
end)

-- เอฟเฟกต์ RGB ขอบ UI
RunService.RenderStepped:Connect(function()
    local color = Color3.fromHSV(tick() % 5 / 5, 0.8, 1)
    Stroke.Color = color
    Title.TextColor3 = color
end)

-- Anti-AFK ป้องกันหลุด
LP.Idled:Connect(function()
    game:GetService("VirtualUser"):CaptureController()
    game:GetService("VirtualUser"):ClickButton2(Vector2.new(0,0))
end)
