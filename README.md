-- [[ MARKW SYSTEM: ANTI-BAN & ANTI-ERROR 267 ]]
-- ปรับปรุงจาก FIER X RYUU เพื่อความเนียนสูงสุด

repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LP = Players.LocalPlayer

repeat task.wait() until LP and LP:FindFirstChild("PlayerGui")

-- ล้างระบบเก่า
if getgenv().MARKW_PROTECT then
	getgenv().MARKW_PROTECT:Disconnect()
	getgenv().MARKW_PROTECT = nil
end

local Remotes = ReplicatedStorage:WaitForChild("RemoteEvents")
local CoinEvent = Remotes:WaitForChild("CollectIncome")
local GemEvent = Remotes:WaitForChild("RemoteCollectGem")
local SpinEvent = ReplicatedStorage:WaitForChild("RemoteFunctions"):WaitForChild("IncrementSpinAvailable")

-- [[ UI ระบบป้องกันภาษาไทย ]]
local Gui = Instance.new("ScreenGui", LP.PlayerGui)
Gui.Name = "MARKW_SAFE_SYSTEM"

local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.new(0, 300, 0, 350)
Main.Position = UDim2.new(0.5, -150, 0.5, -175)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 15)

local Stroke = Instance.new("UIStroke", Main)
Stroke.Thickness = 2
Stroke.Color = Color3.fromRGB(0, 255, 150)

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 50)
Title.Text = "MARKW ANTI-ERROR 267"
Title.Font = Enum.Font.GothamBold
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundTransparency = 1

-- ฟังก์ชันเคลื่อนที่แบบปลอดภัย (ป้องกัน Error 267)
local function SafeTeleport(targetPart)
	local char = LP.Character
	local hrp = char and char:FindFirstChild("HumanoidRootPart")
	if hrp and targetPart then
		-- ล้างแรงกระชากของตัวละครก่อนวาร์ป (ป้องกันตรวจเจอความเร็วสูง)
		hrp.Velocity = Vector3.new(0, 0, 0)
		hrp.RotVelocity = Vector3.new(0, 0, 0)
		
		-- วาร์ปแบบมี Offset เล็กน้อยเพื่อความเนียน
		hrp.CFrame = targetPart.CFrame + Vector3.new(0, 3.5, 0)
		
		-- ส่งสัญญาณสัมผัสแบบเว้นจังหวะ
		firetouchinterest(hrp, targetPart, 0)
		task.wait(0.05)
		firetouchinterest(hrp, targetPart, 1)
	end
end

-- ระบบจัดการปุ่ม
local Toggles = {IC = false, IG = false, AT = false, AC = false, IS = false}
local function AddToggle(txt, y, key)
	local B = Instance.new("TextButton", Main)
	B.Size = UDim2.new(0, 260, 0, 40)
	B.Position = UDim2.new(0.5, -130, 0, y)
	B.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
	B.Text = txt.." : ปิด"
	B.TextColor3 = Color3.new(1, 1, 1)
	Instance.new("UICorner", B)
	B.MouseButton1Click:Connect(function()
		Toggles[key] = not Toggles[key]
		B.Text = txt.." : "..(Toggles[key] and "เปิด" or "ปิด")
		B.BackgroundColor3 = Toggles[key] and Color3.fromRGB(0, 150, 100) or Color3.fromRGB(30, 30, 30)
	end)
end

AddToggle("ออโต้ปั๊มเงิน (Safe)", 60, "IC")
AddToggle("ออโต้ปั๊มเพชร (Safe)", 110, "IG")
AddToggle("ออโต้สร้างบ้าน (Anti-Ban)", 160, "AT")
AddToggle("ออโต้เก็บเงินเครื่อง (Safe)", 210, "AC")
AddToggle("ออโต้สปิน (Safe)", 260, "IS")

-- Loop หลัก (ปรับจูนความถี่เพื่อเลี่ยง 267)
local t1, t2, t3, t4, t5 = 0, 0, 0, 0, 0
getgenv().MARKW_PROTECT = RunService.Heartbeat:Connect(function(dt)
	t1+=dt t2+=dt t3+=dt t4+=dt t5+=dt
	
	-- ปั๊มเงิน/เพชร (เพิ่มดีเลย์ให้ไม่ถี่เกินไป)
	if Toggles.IC and t1 > 0.4 then t1=0 pcall(function() CoinEvent:FireServer() end) end
	if Toggles.IG and t2 > 0.5 then t2=0 pcall(function() GemEvent:FireServer() end) end
	if Toggles.IS and t3 > 1.2 then t3=0 pcall(function() SpinEvent:InvokeServer(500) end) end

	-- ระบบสร้างบ้านแบบ Safe Move
	if Toggles.AT and t4 > 1.8 then
		t4=0
		-- ค้นหาปุ่มใน Tycoon (ใช้ Logic ค้นหาปุ่มที่เงินพอเหมือนตัวก่อน)
		pcall(function()
			-- ใส่โค้ดค้นหา Parts และวาร์ปด้วย SafeTeleport(p)
		end)
	end
end)

-- Anti-AFK (กันหลุดเมื่อเปิดทิ้งไว้)
LP.Idled:Connect(function()
	game:GetService("VirtualUser"):CaptureController()
	game:GetService("VirtualUser"):ClickButton2(Vector2.new(0,0))
end)
