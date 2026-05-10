-- [[ MARKW SYSTEM: GHOST FLYER ]]
-- วิธีใช้: ยืนในบ้านแล้วกดเริ่ม ตัวละครจะลอยไปหาปุ่ม/เหรียญ อย่างนุ่มนวล

repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local TweenService = game:GetService("TweenService")
local Root = LP.Character:WaitForChild("HumanoidRootPart")

_G.GhostFly = false

-- [[ UI สไตล์ MARKW: เรียบหรูกึ่งกลางหน้าจอ ]]
local Gui = Instance.new("ScreenGui", LP.PlayerGui)
local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.new(0, 240, 0, 110)
Main.Position = UDim2.new(0.5, -120, 0.5, -55)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 15)

local Stroke = Instance.new("UIStroke", Main)
Stroke.Thickness = 2
Stroke.Color = Color3.fromRGB(0, 255, 200)

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 45)
Title.Text = "MARKW GHOST FLY"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.GothamBold
Title.BackgroundTransparency = 1

-- [[ ฟังก์ชันลอยตัวแบบนุ่มนวล (Smooth Tween) ]]
local function SmoothFly(targetPos)
    local dist = (Root.Position - targetPos).Magnitude
    if dist > 100 then return end -- ป้องกันลอยออกนอกเขตบ้าน
    
    -- ความเร็ว 18 (เร็วกว่าเดินนิดเดียวแต่ปลอดภัย)
    local speed = 18
    local tweenInfo = TweenInfo.new(dist/speed, Enum.EasingStyle.Linear)
    
    -- ลอยไปที่เป้าหมาย (ยกสูงจากพื้นเล็กน้อยกันติด)
    local tween = TweenService:Create(Root, tweenInfo, {
        CFrame = CFrame.new(targetPos + Vector3.new(0, 2.5, 0))
    })
    
    tween:Play()
    tween.Completed:Wait()
    task.wait(0.5) -- หยุดพักให้เนียน
end

-- [[ ระบบค้นหาเป้าหมาย ]]
task.spawn(function()
    while true do
        if _G.GhostFly then
            pcall(function()
                local target = nil
                local minDist = 60

                for _, v in pairs(workspace:GetDescendants()) do
                    if v:IsA("BasePart") and v.Transparency < 0.5 then
                        local n = v.Name:lower()
                        -- เลือกเฉพาะปุ่มและเหรียญในระยะ
                        if n:find("button") or n:find("pad") or n:find("coin") then
                            local d = (Root.Position - v.Position).Magnitude
                            if d < minDist then
                                minDist = d
                                target = v
                            end
                        end
                    end
                end

                if target then
                    SmoothFly(target.Position)
                    -- ส่งสัญญาณแตะปุ่ม
                    firetouchinterest(Root, target, 0)
                    task.wait(0.1)
                    firetouchinterest(Root, target, 1)
                end
            end)
        end
        task.wait(1)
    end
end)

-- ปุ่มเปิดโหมดลอย
local Btn = Instance.new("TextButton", Main)
Btn.Size = UDim2.new(0, 200, 0, 45)
Btn.Position = UDim2.new(0.5, -100, 0, 50)
Btn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Btn.Text = "เริ่มการลอยฟาร์ม"
Btn.TextColor3 = Color3.new(1, 1, 1)
Btn.Font = Enum.Font.GothamBold
Instance.new("UICorner", Btn)

Btn.MouseButton1Click:Connect(function()
    _G.GhostFly = not _G.GhostFly
    Btn.Text = _G.GhostFly and "กำลังลอย..." or "เริ่มการลอยฟาร์ม"
    Btn.BackgroundColor3 = _G.GhostFly and Color3.fromRGB(0, 150, 100) or Color3.fromRGB(30, 30, 30)
end)
