-- StarterGui/FrostzGui/Main.client.lua
-- GUI "Frostz Tween+" com botões, toggle e animações

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

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

-- drag helper (não muda visual)
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

local gui = script.Parent :: ScreenGui
gui.IgnoreGuiInset, gui.ResetOnSpawn = true, false

-- janela
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

	local on=false
	local function set(v)
		on=v; toggle.Text=v and "ON" or "OFF"
		tween(toggle,{BackgroundColor3=v and Colors.Pink or Color3.fromRGB(30,30,36)},.15):Play()
		tween(knob,{Position=v and UDim2.fromOffset(38,2) or UDim2.fromOffset(2,2)},.15):Play()
	end
	toggle.MouseButton1Click:Connect(function() set(not on) end)
	return frame, set, function() return on end, toggle
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
local autoKickFrame, setAutoKick, getAutoKick = (function()
	local f, set, get, btn = makeToggle(body,"AUTO KICK")
	return f, set, get
end)()
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
	local on = false
	local function set(v)
		on = v; b.Text = v and "ON" or "OFF"
		tween(b, {BackgroundColor3 = v and Colors.Pink or Color3.fromRGB(30,30,36)}, .14):Play()
		tween(knob, {Position = v and UDim2.fromOffset(38,2) or UDim2.fromOffset(2,2)}, .14):Play()
	end
	b.MouseButton1Click:Connect(function() set(not on) end)
	return {Frame=f, Set=set, Get=function() return on end, Button=b}
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

-- ======= Drag nas DUAS janelas =======
makeDraggable(window)
makeDraggable(sub)

-- ====================================================================
-- =====================  LÓGICA TELEGUIADO / KICK  ====================
-- ====================================================================

local LocalPlayer = Players.LocalPlayer

-- >>> tenta localizar a base. Ajuste os nomes conforme seu mapa.
local function findBaseCFrame()
	-- 1) Base do jogador por nome
	local candidates = {
		("Base_%s"):format(LocalPlayer.Name),
		("Base-%s"):format(LocalPlayer.Name),
		("Base%s"):format(LocalPlayer.Name),
		"Base", "BASE", "HomeBase", "PlayerBase"
	}
	for _, name in ipairs(candidates) do
		local obj = workspace:FindFirstChild(name)
		if obj and obj:IsA("BasePart") then
			return obj.CFrame
		elseif obj and obj:IsA("Model") and obj.PrimaryPart then
			return obj.PrimaryPart.CFrame
		end
	end
	-- 2) Pasta Bases -> modelo/nó do jogador
	local folder = workspace:FindFirstChild("Bases") or workspace:FindFirstChild("PlayerBases")
	if folder then
		local mine = folder:FindFirstChild(LocalPlayer.Name) or folder:FindFirstChild(("Base_%s"):format(LocalPlayer.Name))
		if mine then
			if mine:IsA("BasePart") then return mine.CFrame end
			if mine:IsA("Model") and mine.PrimaryPart then return mine.PrimaryPart.CFrame end
			-- pega primeiro BasePart dentro
			for _, d in ipairs(mine:GetDescendants()) do
				if d:IsA("BasePart") then return d.CFrame end
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

-- >>> “usar” uma Taser no próprio boneco (se existir uma Tool com esse nome)
local function taserSelf()
	local char = LocalPlayer.Character
	if not char then return end
	local toolNames = {"Taser", "TaserGun", "Taser Gun", "Stun", "StunGun"}
	local tool
	-- procura no Character
	for _, n in ipairs(toolNames) do
		tool = char:FindFirstChild(n)
		if tool and tool:IsA("Tool") then break end
	end
	-- se não achou, procura no Backpack
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
	if tool and tool:IsA("Tool") and tool:FindFirstChild("Handle") then
		-- ativa (visual/efeito do Tool, se existir)
		pcall(function() tool:Activate() end)
	end
	-- efeito básico: breves tremores/“taser”
	local _, hum, hrp = getCharStuff()
	if hum then
		pcall(function() hum:ChangeState(Enum.HumanoidStateType.FallingDown) end)
	end
	if hrp then
		local t0 = tick()
		RunService.Heartbeat:Wait()
		while tick() - t0 < 0.25 do
			hrp.CFrame = hrp.CFrame * CFrame.Angles(0, 0, math.rad(3))
			RunService.Heartbeat:Wait()
		end
	end
end

-- >>> voar diretamente até a base
local currentFlight -- tween/loop cancel token
local function flyStraightTo(cfTarget, speed)
	speed = speed or 120 -- studs/seg
	local char, hum, hrp = getCharStuff()
	if not (hum and hrp and cfTarget) then return end

	-- cancela voo anterior
	if currentFlight then currentFlight.cancelled = true end
	currentFlight = {cancelled = false}

	-- “voando”: vamos ancorar para deslocar suave (simples e direto)
	local prevAnchored = hrp.Anchored
	local prevAutoRotate = hum.AutoRotate
	hum.AutoRotate = false
	hrp.Anchored = true

	local arrived = false
	local heartbeat = RunService.Heartbeat
	local function dist(a, b) return (a - b).Magnitude end

	while not currentFlight.cancelled do
		local pos = hrp.Position
		local goal = cfTarget.Position + Vector3.new(0, 5, 0) -- chegue 5 studs acima
		local d = goal - pos
		local distance = d.Magnitude
		if distance < 6 then
			arrived = true
			break
		end
		local dt = heartbeat:Wait()
		local step = math.min(distance, speed * dt)
		hrp.CFrame = CFrame.new(pos + d.Unit * step, goal) -- olha pro alvo
	end

	-- restaurar
	hrp.Anchored = prevAnchored
	hum.AutoRotate = prevAutoRotate

	-- chegou? dispara anti-kick se ativo
	if arrived and getAutoKick and getAutoKick() then
		-- pequena graça visual antes do kick
		task.delay(0.1, function()
			pcall(function()
				LocalPlayer:Kick("voce coletou brainrot com sucesso")
			end)
		end)
	end
end

-- >>> pipeline do Teleguiado: taser-se + voar pra base
local teleRunning = false
local function runTeleGuiado()
	if teleRunning then return end
	teleRunning = true
	-- 1) taser self
	taserSelf()
	-- 2) achar base
	local baseCF = findBaseCFrame()
	if not baseCF then
		warn("[Teleguiado] Base não encontrada. Ajuste os nomes em findBaseCFrame().")
		teleRunning = false
		return
	end
	-- 3) voar até a base
	flyStraightTo(baseCF, 140)
	teleRunning = false
end

-- ======= callbacks UI =======
espGod.MouseButton1Click:Connect(function() end)
espSecret.MouseButton1Click:Connect(function() end)
espBase.MouseButton1Click:Connect(function() end)
espPlayer.MouseButton1Click:Connect(function() end)

-- abrir/fechar sub-gui
-- (já conectado acima)

-- ligar/desligar TELEGUIADO pela guizinha
tTele.Button.MouseButton1Click:Connect(function()
	-- o próprio toggle já alterna visual; aqui a gente dispara a ação se ficou ON
	task.defer(function()
		if tTele.Get() then
			runTeleGuiado()
		else
			-- cancelar voo se estava em andamento
			if currentFlight then currentFlight.cancelled = true end
		end
	end)
end)

-- Aimbot: deixei sem lógica (você disse apenas o nome). Se quiser, implemento depois.
tAimb.Button.MouseButton1Click:Connect(function()
	-- lugar certo pra ativar/desativar seu aimbot
end)

-- estados iniciais
setAutoKick(false)
