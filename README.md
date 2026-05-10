repeat task.wait() until game:IsLoaded()

local Players=game:GetService("Players")
local RunService=game:GetService("RunService")
local ReplicatedStorage=game:GetService("ReplicatedStorage")

local LP=Players.LocalPlayer
repeat task.wait() until LP and LP:FindFirstChild("PlayerGui")

local PlayerGui=LP.PlayerGui

if getgenv().FIERXRYUU then
	getgenv().FIERXRYUU:Disconnect()
	getgenv().FIERXRYUU=nil
end

local Remotes=ReplicatedStorage:WaitForChild("RemoteEvents")
local CoinEvent=Remotes:WaitForChild("CollectIncome")
local GemEvent=Remotes:WaitForChild("RemoteCollectGem")
local SpinEvent=ReplicatedStorage:WaitForChild("RemoteFunctions"):WaitForChild("IncrementSpinAvailable")

local BAD={"robux","upgrade","dev","product","premium","gamepass","donate"}

local function IsBad(obj)
	local cur=obj
	while cur do
		local n=cur.Name:lower()
		for _,b in ipairs(BAD) do
			if n:find(b) then return true end
		end
		cur=cur.Parent
	end
	for _,d in pairs(obj:GetDescendants()) do
		local n=d.Name:lower()
		for _,b in ipairs(BAD) do
			if n:find(b) then return true end
		end
	end
	return false
end

local function GetTycoon()
	local char=LP.Character
	if not char or not char:FindFirstChild("HumanoidRootPart") then return end
	local hrp=char.HumanoidRootPart
	local best=nil
	local dist=math.huge
	for _,v in pairs(workspace:GetChildren()) do
		if v:IsA("Model") and v.Name:lower():find("tycoon") then
			local p=v:FindFirstChildWhichIsA("BasePart",true)
			if p then
				local d=(hrp.Position-p.Position).Magnitude
				if d<dist then
					dist=d
					best=v
				end
			end
		end
	end
	return best
end

local MyTycoon=nil
local Parts={}
local Collectors={}

task.spawn(function()
	while true do
		MyTycoon=GetTycoon()
		task.wait(0.5)
	end
end)

local function Refresh()
	Parts={}
	Collectors={}
	if MyTycoon then
		for _,v in pairs(MyTycoon:GetDescendants()) do
			if v:IsA("BasePart") and not IsBad(v) then
				local n=v.Name:lower()
				if n:find("button") or n:find("pad") or n:find("drop") then
					table.insert(Parts,v)
				end
				if n:find("collector") or n:find("collider") then
					table.insert(Collectors,v)
				end
			end
		end
	end
end

task.spawn(function()
	while true do
		Refresh()
		task.wait(1)
	end
end)

local Gui=Instance.new("ScreenGui",PlayerGui)
Gui.Name="FIER_PREMIUM"
Gui.ResetOnSpawn=false

local Open=Instance.new("TextButton",Gui)
Open.Size=UDim2.new(0,150,0,40)
Open.Position=UDim2.new(0,15,0.5,-20)
Open.Text="FIER HUB"
Open.BackgroundColor3=Color3.fromRGB(20,20,20)
Open.TextColor3=Color3.new(1,1,1)
Instance.new("UICorner",Open)

local Main=Instance.new("Frame",Gui)
Main.Size=UDim2.new(0,460,0,320)
Main.Position=UDim2.new(0.5,-230,0.5,-160)
Main.BackgroundColor3=Color3.fromRGB(15,15,15)
Main.Visible=false
Main.Active=true
Instance.new("UICorner",Main)

local Stroke=Instance.new("UIStroke",Main)
Stroke.Thickness=3

local Top=Instance.new("Frame",Main)
Top.Size=UDim2.new(1,0,0,50)
Top.BackgroundColor3=Color3.fromRGB(25,25,25)
Instance.new("UICorner",Top)

local Title=Instance.new("TextLabel",Top)
Title.Size=UDim2.new(1,-60,1,0)
Title.Position=UDim2.new(0,20,0,0)
Title.BackgroundTransparency=1
Title.Text="FIER X RYUU PREMIUM"
Title.Font=Enum.Font.GothamBlack
Title.TextSize=16
Title.TextXAlignment=Enum.TextXAlignment.Left

local Close=Instance.new("TextButton",Top)
Close.Size=UDim2.new(0,30,0,30)
Close.Position=UDim2.new(1,-40,0.5,-15)
Close.Text="X"
Close.BackgroundColor3=Color3.fromRGB(40,40,40)
Close.TextColor3=Color3.new(1,1,1)
Instance.new("UICorner",Close)

local function rgb()
	return Color3.fromHSV((tick()*0.2)%1,1,1)
end

RunService.RenderStepped:Connect(function()
	local c=rgb()
	Stroke.Color=c
	Title.TextColor3=c
end)

local function Toggle(txt,y,cb)
	local B=Instance.new("TextButton",Main)
	B.Size=UDim2.new(0,260,0,35)
	B.Position=UDim2.new(0,20,0,y)
	B.Text=txt.." : OFF"
	B.BackgroundColor3=Color3.fromRGB(30,30,30)
	B.TextColor3=Color3.new(1,1,1)
	local s=false
	B.MouseButton1Click:Connect(function()
		s=not s
		B.Text=txt.." : "..(s and "ON" or "OFF")
		cb(s)
	end)
end

local IC=false
local IG=false
local AT=false
local AC=false
local IS=false

Toggle("Infinity Coins",60,function(v) IC=v end)
Toggle("Infinity Gems",110,function(v) IG=v end)
Toggle("Auto Tycoon",160,function(v) AT=v end)
Toggle("Auto Collect (1s TP)",210,function(v) AC=v end)
Toggle("Infinity Spins",260,function(v) IS=v end)

local i=1
local ci=1

local function Step()
	local p=Parts[i]
	if p and LP.Character and LP.Character:FindFirstChild("HumanoidRootPart") then
		local hrp=LP.Character.HumanoidRootPart
		hrp.CFrame=p.CFrame+Vector3.new(0,3,0)
		firetouchinterest(hrp,p,0)
		firetouchinterest(hrp,p,1)
	end
	i=i+1
	if i>#Parts then i=1 end
end

local function CollectTP()
	local p=Collectors[ci]
	if p and LP.Character and LP.Character:FindFirstChild("HumanoidRootPart") then
		local hrp=LP.Character.HumanoidRootPart
		hrp.CFrame=p.CFrame+Vector3.new(0,3,0)
		firetouchinterest(hrp,p,0)
		firetouchinterest(hrp,p,1)
	end
	ci=ci+1
	if ci>#Collectors then ci=1 end
end

local t1,t2,t3,t4,t5=0,0,0,0,0

getgenv().FIERXRYUU=RunService.Heartbeat:Connect(function(dt)
	t1+=dt t2+=dt t3+=dt t4+=dt t5+=dt

	if IC and t1>0.06 then t1=0 pcall(function() CoinEvent:FireServer() end) end
	if IG and t2>0.05 then t2=0 for i=1,6 do pcall(function() GemEvent:FireServer() end) end end
	if IS and t3>0.1 then t3=0 pcall(function() SpinEvent:InvokeServer(100000) end) end

	if AT and t4>0.01 then
		t4=0
		pcall(function() Step() end)
	end

	if AC and t5>1 then
		t5=0
		pcall(function() CollectTP() end)
	end
end)

Open.MouseButton1Click:Connect(function()
	Main.Visible=true
	Open.Visible=false
end)

Close.MouseButton1Click:Connect(function()
	Main.Visible=false
	Open.Visible=true
end)
