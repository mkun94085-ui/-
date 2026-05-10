-- [[ MARKW SYSTEM: THE PATHFINDER ]]
-- วิธีใช้: ยืนในบ้านตัวเองแล้วกดเริ่ม ระบบจะเดินไปซื้อปุ่มที่ใกล้ที่สุดเอง

repeat task.wait() until game:IsLoaded()

local PathfindingService = game:GetService("PathfindingService")
local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local Character = LP.Character or LP.CharacterAdded:Wait()
local Hum = Character:WaitForChild("Humanoid")
local Root = Character:WaitForChild("HumanoidRootPart")

_G.GhostWalk = false

-- [[ UI สไตล์ MARKW: เรียบง่ายและอยู่กึ่งกลาง ]]
local Gui = Instance.new("ScreenGui", LP.PlayerGui)
local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.new(0, 240, 0, 100)
Main.Position = UDim2.new(0.5, -120, 0.5, -50)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 15)

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Text = "MARKW PATHFINDER"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.BackgroundTransparency = 1

-- [[ ฟังก์ชันเดินแบบฉลาด (หลบกำแพง) ]]
local function SmartWalk(targetPos)
    local path = PathfindingService:CreatePath({
        AgentRadius = 2,
        AgentHeight = 5,
        AgentCanJump = true,
        AgentJumpHeight = 10,
    })
    
    local success, errorMessage = pcall(function()
        path:ComputeAsync(Root.Position, targetPos)
    end)

    if success and path.Status == Enum.PathStatus.Success then
        local waypoints = path:GetWaypoints()
        for _, waypoint in pairs(waypoints) do
            if not _G.GhostWalk then break end
            
            -- ถ้าต้องกระโดด
            if waypoint.Action == Enum.PathWaypointAction.Jump then
                Hum.Jump = true
            end
            
            Hum:MoveTo(waypoint.Position)
            -- รอจนกว่าจะถึงจุดพักย่อย (เนียนมาก)
            local timeOut = 0
            while (Root.Position - waypoint.Position).Magnitude > 2 and timeOut < 10 do
                task.wait(0.1)
                timeOut = timeOut + 1
            end
        end
    else
        -- ถ้า Pathfinding ล้มเหลว ให้เดินตรงๆ ไปเลย (กรณีระยะสั้น)
        Hum:MoveTo(targetPos)
        Hum.MoveToFinished:Wait()
    end
end

-- [[ ระบบสแกนหาปุ่มและเหรียญ ]]
task.spawn(function()
    while true do
        if _G.GhostWalk then
            pcall(function()
                local target = nil
                local minDist = 50 -- เดินในระยะ 50 เมตรเพื่อความปลอดภัย

                -- ค้นหาปุ่มที่สร้างได้
                for _, v in pairs(workspace:GetDescendants()) do
                    if v:IsA("BasePart") and v.Transparency < 0.5 then
                        local n = v.Name:lower()
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
                    SmartWalk(target.Position)
                    task.wait(math.random(1, 2)) -- หยุดรอหลังถึงจุดหมาย เหมือนคนเล่นจริง
                end
            end)
        end
        task.wait(1)
    end
end)

-- ปุ่มเปิดโหมด
local Btn = Instance.new("TextButton", Main)
Btn.Size = UDim2.new(0, 200, 0, 40)
Btn.Position = UDim2.new(0.5, -100, 0, 45)
Btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Btn.Text = "เริ่มเดินฟาร์ม (ปลอดภัย)"
Btn.TextColor3 = Color3.new(1, 1, 1)
Btn.Font = Enum.Font.GothamBold
Instance.new("UICorner", Btn)

Btn.MouseButton1Click:Connect(function()
    _G.GhostWalk = not _G.GhostWalk
    Btn.Text = _G.GhostWalk and "กำลังเดิน..." or "เริ่มเดินฟาร์ม (ปลอดภัย)"
    Btn.BackgroundColor3 = _G.GhostWalk and Color3.fromRGB(0, 150, 100) or Color3.fromRGB(40, 40, 40)
end)

-- Anti-AFK
LP.Idled:Connect(function()
    game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    task.wait(1)
    game:GetService("VirtualUser"):Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
end)
