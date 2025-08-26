-- StarterGui/FrostzGui/Main.client.lua
-- GUI "Frostz Tween+" com botões, toggle, subpainel Teleguiado e animações

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local KickRE = ReplicatedStorage:WaitForChild("FrostzKick") -- RemoteEvent criado no servidor

--======================== helpers UI ========================--
local function tween(o, props, t, style, dir)
	return TweenService:Create(o, TweenInfo.new(t or 0.2, style or Enum.EasingStyle.Quint, dir or Enum.EasingDirection.Out), props)
end
local function mk(parent, class, props)
	local ins = Instance.new(class)
	for k, v in pairs(props or {}) do ins[k] = v end
	ins.Parent = parent
	return ins
end
local function mkCorner(parent, r) mk(parent, "UICorner", {CornerRadius = UDim.new(0, r or 12)}) end
local function mkStroke(parent, c, th) mk(parent, "UIStroke", {Color = c or Color3.fromRGB(255,214,10), Thickness = th or 3, ApplyStrokeMode = Enum.ApplyStrokeMode.Border}) end

-- Drag sem mudar visual
local function makeDraggable(frame, dragArea)
	dragArea = dragArea or frame
	local dragging, dragStart, startPos
	dragArea.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = frame.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then dragging = false end
			end)
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			local delta = input.Position - dragStart
			frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		end
	end)
end

--======================== raiz GUI ========================--
local gui = script.Parent :: ScreenGui
gui.IgnoreGuiInset, gui.ResetOnSpawn = true, false

-- janela principal
local window = mk(gui, "Frame", {
	Name="Window", Position=UDim2.fromOffset(40,60), Size=UDim2.fromOffset(290,360),
	BackgroundColor3=Color3.fromRGB(33,33,40)
})
mkCorner(window,16); mkStroke(window, Color3.fromRGB(255,204,0), 4)
mk(window,"ImageLabel",{BackgroundTransparency=1,Image="rbxassetid://5028857084",ImageTransparency=.35,
	ScaleType=Enum.ScaleType.Slice, SliceCenter=Rect.new(24,24,276,276), Size=UDim2.fromScale(1,1), Position=UDim2.fromOffset(0,6)})

-- cabeçalho
local header = mk(window,"Frame",{Size=UDim2.new(1,-16,0,64),Position=UDim2.fromOffset(8,8),BackgroundColor3=Color3.fromRGB(46,46,56)})
mkCorner(header,12); mkStroke(header, Color3.fromRGB(255,204,0), 2)
mk(header,"TextLabel",{BackgroundTransparency=1,Size=UDim2.new(1,-16,0,28),Position=UDim2.fromOffset(8,6),
	Font=Enum.Font.GothamBlack, Text="Frostz Tween+", TextSize=18, TextXAlignment=Enum.TextXAlignment.Left,
	TextColor3=Color3.fromRGB(255,105,130)})
mk(header,"TextLabel",{BackgroundTransparency=1,Size=UDim2.new(1,-16,0,22),Position=UDim2.fromOffset(8,34),
	Font=Enum.Font.GothamMedium, Text="TIKTOK: @seuuseraqui", TextSize=14, TextXAlignment=Enum.TextXAlignment.Left,
	TextColor3=Color3.fromRGB(230,230,230)})

-- corpo
local body = mk(window,"Frame",{BackgroundTransparency=1,Position=UDim2.fromOffset(8,80),Size=UDim2.new(1,-16,1,-88)})
mk(body,"UIListLayout",{Padding=UDim.new(0,10),FillDirection=Enum.FillDirection.Vertical,SortOrder=Enum.SortOrder.LayoutOrder})

-- paleta
local Colors = {
	Pink=Color3.fromRGB(235,61,115), PinkDark=Color3.fromRGB(200,42,90),
	Red=Color3.fromRGB(235,47,55), RedDark=Color3.fromRGB(190,36,45),
	Gray=Color3.fromRGB(55,55,65), Text=Color3.fromRGB(245,245,245)
}

local function makeButton(parent, text, a, b)
	local btn = mk(parent,"TextButton",{Text=text,Font=Enum.Font.GothamBold,TextSize=18,TextColor3=Colors.Text,
		AutoButtonColor=false,BackgroundColor3=a,Size=UDim2.new(1,0,0,44)})
	mkCorner(btn,12); mk(btn,"UIGradient",{Rotation=90,Color=ColorSequence.new{ColorSequenceKeypoint.new(0,a),ColorSequenceKeypoint.new(1,b)}})
	btn.MouseEnter:Connect(function() tween(btn,{BackgroundColor3=b},.12):Play() end)
	btn.MouseLeave:Connect(function() tween(btn,{BackgroundColor3=a},.12):Play() end)
	return btn
end

local function makeToggle(parent, labelText)
	local frame = mk(parent,"Frame",{BackgroundColor3=Colors.Gray,Size=UDim2.new(1,0,0,44)}); mkCorner(frame,12)
	mk(frame,"TextLabel",{BackgroundTransparency=1,Size=UDim2.new(1,-70,1,0),Position=UDim2.fromOffset(12,0),
		Font=Enum.Font.GothamBold,Text=labelText,TextSize=18,TextXAlignment=Enum.TextXAlignment.Left,TextColor3=Colors.Text})
	local toggle = mk(frame,"TextButton",{AnchorPoint=Vector2.new(1,.5),Position=UDim2.new(1,-10,.5,0),Size=UDim2.fromOffset(64,28),
		BackgroundColor3=Color3.fromRGB(30,30,36),AutoButtonColor=false,Text="OFF",Font=Enum.Font.GothamBold,TextSize=14,TextColor3=Color3.new(1,1,1)})
	mkCorner(toggle,14)
	local knob = mk(toggle,"Frame",{Size=UDim2.fromOffset(24,24),Position=UDim2.fromOffset(2,2),BackgroundColor3=Color3.fromRGB(120,120,125)})
	mkCorner(knob,12)

	local evt = Instance.new("BindableEvent")
	local on=false
	local function set(v)
		on=v; toggle.Text=v and "ON" or "OFF"
		tween(toggle,{BackgroundColor3=v and Colors.Pink or Color3.fromRGB(30,30,36)},.15):Play()
		tween(knob,{Position=v and UDim2.fromOffset(38,2) or UDim2.fromOffset(2,2)},.15):Play()
		evt:Fire(on)
	end
	toggle.MouseButton1Click:Connect(function() set(not on) end)
	return frame, set, function() return on end, evt.Event
end

-- linha dupla (ESP GOD / ESP SECRET)
local row = mk(body,"Frame",{BackgroundTransparency=1,Size=UDim2.new(1,0,0,44)})
mk(row,"UIListLayout",{FillDirection=Enum.FillDirection.Horizontal,Padding=UDim.new(0,10)})
local espGod = makeButton(row,"ESP GOD", Colors.Pink, Colors.PinkDark); espGod.Size = UDim2.new(0.5,-5,1,0)
local espSecret = makeButton(row,"ESP SECRET", Colors.Gray, Color3.fromRGB(45,45,55)); espSecret.Size = UDim2.new(0.5,-5,1,0)

-- botões únicos
local espBase   = makeButton(body,"ESP BASE",   Colors.Pink, Colors.PinkDark)
local espPlayer = makeButton(body,"ESP PLAYER", Colors.Gray, Color3.fromRGB(45,45,55))

-- toggle + botão final
local autoKickFrame, setAutoKick, getAutoKick, onAutoKickChanged = makeToggle(body,"AUTO KICK")
local painel = makeButton(body,"Painel Teleguiado", Colors.Red, Colors.RedDark)

-- ========= Sub-GUI: Painel Teleguiado =========
local sub = mk(gui, "Frame", {
	Name = "TelePanel",
	Visible = false,
	BackgroundColor3 = Color3.fromRGB(32,32,38),
	Size = UDim2.fromOffset(220, 140),
	Position = window.Position + UDim2.fromOffset(window.Size.X.Offset + 10, 0)
})
mkCorner(sub, 12); mkStroke(sub, Color3.fromRGB(255,204,0), 2)

mk(sub, "TextLabel", {
	BackgroundTransparency = 1,
	Text = "Painel Teleguiado",
	Font = Enum.Font.GothamBold,
	TextSize = 16,
	TextColor3 = Colors.Text,
	Size = UDim2.new(1, -12, 0, 26),
	Position = UDim2.fromOffset(6, 6),
	TextXAlignment = Enum.TextXAlignment.Left
})

local subBody = mk(sub, "Frame", {BackgroundTransparency = 1, Position = UDim2.fromOffset(8, 38), Size = UDim2.new(1,-16,1,-46)})
mk(subBody, "UIListLayout", {Padding = UDim.new(0,8), FillDirection = Enum.FillDirection.Vertical})

local function makeMiniToggle(parent, text)
	local f = mk(parent, "Frame", {BackgroundColor3 = Colors.Gray, Size = UDim2.new(1,0,0,36)}); mkCorner(f, 10)
	mk(f, "TextLabel", {BackgroundTransparency=1, Text=text, Font=Enum.Font.GothamBold, TextSize=15, TextColor3=Colors.Text,
		Size=UDim2.new(1,-70,1,0), Position=UDim2.fromOffset(10,0), TextXAlignment=Enum.TextXAlignment.Left})
	local b = mk(f, "TextButton", {AnchorPoint=Vector2.new(1,.5), Position=UDim2.new(1,-8,.5,0), Size=UDim2.fromOffset(60,24),
		BackgroundColor3=Color3.fromRGB(30,30,36), AutoButtonColor=false, Text="OFF", Font=Enum.Font.GothamBold, TextSize=13, TextColor3=Colors.Text})
	mkCorner(b, 12)
	local knob = mk(b, "Frame", {Size=UDim2.fromOffset(20,20), Position=UDim2.fromOffset(2,2), BackgroundColor3=Color3.fromRGB(120,120,125)}); mkCorner(knob,10)

	local evt = Instance.new("BindableEvent")
	local on = false
	local function set(v)
		on = v; b.Text = v and "ON" or "OFF"
		tween(b, {BackgroundColor3 = v and Colors.Pink or Color3.fromRGB(30,30,36)}, .14):Play()
		tween(knob, {Position = v and UDim2.fromOffset(38,2) or UDim2.fromOffset(2,2)}, .14):Play()
		evt:Fire(on)
	end
	b.MouseButton1Click:Connect(function() set(not on) end)
	return {Frame=f, Set=set, Get=function() return on end, Changed=evt.Event}
end

local tTele = makeMiniToggle(subBody, "TELEGUIADO")
local tAimb = makeMiniToggle(subBody, "AIMBOT")

-- abrir/fechar sub-gui ao clicar no botão "Painel Teleguiado"
painel.MouseButton1Click:Connect(function()
	sub.Visible = not sub.Visible
	if sub.Visible then
		sub.Position = window.Position + UDim2.fromOffset(window.Size.X.Offset + 10, 0)
	end
end)

-- Drag nas duas janelas
makeDraggable(window)
makeDraggable(sub)

--======================== Lógica Teleguiado / Kick ========================--
local LocalPlayer = Players.LocalPlayer

-- ACHAR BASE: ajuste aqui se sua base tiver nome específico
local function findBaseCFrame()
	-- candidatos comuns
	local candidates = {
		("Base_%s"):format(LocalPlayer.Name),
		("Base-%s"):format(LocalPlayer.Name),
		("Base%s"):format(LocalPlayer.Name),
		"Base","BASE","HomeBase","PlayerBase"
	}
	for _, name in ipairs(candidates) do
		local obj = workspace:FindFirstChild(name)
		if obj then
			if obj:IsA("BasePart") then return obj.CFrame end
			if obj:IsA("Model") then
				if obj.PrimaryPart then return obj.PrimaryPart.CFrame end
				for _, d in ipairs(obj:GetDescendants()) do
					if d:IsA("BasePart") then return d.CFrame end
				end
			end
		end
	end
	-- pasta Bases/PlayerBases
	local folder = workspace:FindFirstChild("Bases") or workspace:FindFirstChild("PlayerBases")
	if folder then
		local mine = folder:FindFirstChild(LocalPlayer.Name) or folder:FindFirstChild(("Base_%s"):format(LocalPlayer.Name))
		if mine then
			if mine:IsA("BasePart") then return mine.CFrame end
			if mine:IsA("Model") then
				if mine.PrimaryPart then return mine.PrimaryPart.CFrame end
				for _, d in ipairs(mine:GetDescendants()) do
					if d:IsA("BasePart") then return d.CFrame end
				end
			end
		end
	end
	return nil
end

local function getCharStuff()
	local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
	local hum = char:FindFirstChildOfClass("Humanoid")
	local hrp = char:FindFirstChild("HumanoidRootPart")
	return char, hum, hrp
end

-- “usar” Taser em si mesmo (se existir)
local function taserSelf()
	local char = LocalPlayer.Character
	if not char then return end
	local toolNames = {"Taser","TaserGun","Taser Gun","Stun","StunGun"}
	local tool

	-- procura no Character
	for _, n in ipairs(toolNames) do
		local t = char:FindFirstChild(n)
		if t and t:IsA("Tool") then tool = t break end
	end
	-- ou no Backpack
	if not tool then
		local bp = LocalPlayer:FindFirstChildOfClass("Backpack")
		if bp then
			for _, n in ipairs(toolNames) do
				local t = bp:FindFirstChild(n)
				if t and t:IsA("Tool") then tool = t break end
			end
			if tool then tool.Parent = char end
		end
	end

	if tool and tool:IsA("Tool") then
		pcall(function() tool:Activate() end)
	end

	-- efeito rapidinho (tremor)
	local _, hum, hrp = getCharStuff()
	if hum then pcall(function() hum:ChangeState(Enum.HumanoidStateType.FallingDown) end) end
	if hrp then
		local t0 = tick()
		RunService.Heartbeat:Wait()
		while tick() - t0 < 0.25 do
			hrp.CFrame = hrp.CFrame * CFrame.Angles(0, 0, math.rad(3))
			RunService.Heartbeat:Wait()
		end
	end
end

-- voo reto até a base
local currentFlight
local function flyStraightTo(cfTarget, speed)
	speed = speed or 140
	local char, hum, hrp = getCharStuff()
	if not (hum and hrp and cfTarget) then return end

	-- cancela voo anterior
	if currentFlight then currentFlight.cancelled = true end
	currentFlight = {cancelled = false}

	local prevAnchored = hrp.Anchored
	local prevAutoRotate = hum.AutoRotate
	hum.AutoRotate = false
	hrp.Anchored = true

	local arrived = false
	local function distance(a, b) return (a - b).Magnitude end

	while not currentFlight.cancelled do
		local pos = hrp.Position
		local goal = cfTarget.Position + Vector3.new(0, 5, 0)
		local d = goal - pos
		local dist = d.Magnitude
		if dist < 6 then
			arrived = true
			break
		end
		local dt = RunService.Heartbeat:Wait()
		local step = math.min(dist, speed * dt)
		hrp.CFrame = CFrame.new(pos + d.Unit * step, goal)
	end

	-- restaurar
	hrp.Anchored = prevAnchored
	hum.AutoRotate = prevAutoRotate

	-- chegou na base? dispara kick no servidor se AutoKick ON
	if arrived and getAutoKick and getAutoKick() then
		KickRE:FireServer()
	end
end

-- pipeline do Teleguiado
local teleRunning = false
local function runTeleguiado()
	if teleRunning then return end
	teleRunning = true
	taserSelf()
	local baseCF = findBaseCFrame()
	if not baseCF then
		warn("[Teleguiado] Base não encontrada. Ajuste os nomes em findBaseCFrame().")
		teleRunning = false
		return
	end
	flyStraightTo(baseCF, 140)
	teleRunning = false
end

--=============== callbacks UI ===============--
espGod.MouseButton1Click:Connect(function() end)
espSecret.MouseButton1Click:Connect(function() end)
espBase.MouseButton1Click:Connect(function() end)
espPlayer.MouseButton1Click:Connect(function() end)

-- abrir/fechar sub-gui
painel.MouseButton1Click:Connect(function()
	sub.Visible = not sub.Visible
	if sub.Visible then
		sub.Position = window.Position + UDim2.fromOffset(window.Size.X.Offset + 10, 0)
	end
end)

-- TELEGUIADO: quando o toggle mudar
tTele.Changed:Connect(function(on)
	if on then
		runTeleguiado()
	else
		if currentFlight then currentFlight.cancelled = true end
	end
end)

-- AIMBOT: pronto para ligar/desligar se você quiser implementar depois
tAimb.Changed:Connect(function(on)
	-- ligar/desligar sua lógica de AIMBOT aqui
end)

-- estados iniciais
setAutoKick(false)
