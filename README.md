-- Painel de testes para o SEU jogo (LocalScript em StarterPlayerScripts)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local player = Players.LocalPlayer
if not player then warn("[PainelTeste] Use um LocalScript, não um Script.") return end
local camera = workspace.CurrentCamera
local pgui = player:WaitForChild("PlayerGui")

local old = pgui:FindFirstChild("PainelTeste")
if old then old:Destroy() end

-- ================= ESTADO =================
local state = {
	esp = false, speed = false, jump = false, noclip = false,
	aim = false, sky = false,
	speedValue = 50, fov = 120, aimPart = "Cabeça", smooth = 25,
}
local DEFAULT_SPEED, DEFAULT_JUMP, SUPER_JUMP = 16, 50, 120
local connections = {}
local function bind(c) table.insert(connections, c) return c end

local function getChar() return player.Character end
local function getHum() local c = getChar() return c and c:FindFirstChildOfClass("Humanoid") end
local function getHRP() local c = getChar() return c and c:FindFirstChild("HumanoidRootPart") end

-- ================= INTERFACE =================
local gui = Instance.new("ScreenGui")
gui.Name = "PainelTeste"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = pgui

local main = Instance.new("Frame")
main.Size = UDim2.new(0, 230, 0, 300)
main.Position = UDim2.new(0, 15, 0.5, -150)
main.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
main.BorderSizePixel = 0
main.Active = true
main.Parent = gui
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 10)
local limit = Instance.new("UISizeConstraint", main)
limit.MaxSize = Vector2.new(230, 300)

-- encolhe em telas baixas (iPhone deitado)
local function fitScreen()
	local h = camera.ViewportSize.Y
	main.Size = UDim2.new(0, 230, 0, math.clamp(h - 40, 150, 300))
end
fitScreen()
bind(camera:GetPropertyChangedSignal("ViewportSize"):Connect(fitScreen))

local top = Instance.new("TextLabel")
top.Size = UDim2.new(1, -40, 0, 32)
top.Position = UDim2.new(0, 10, 0, 0)
top.BackgroundTransparency = 1
top.Text = "Painel de Teste"
top.TextColor3 = Color3.new(1, 1, 1)
top.Font = Enum.Font.GothamBold
top.TextSize = 15
top.TextXAlignment = Enum.TextXAlignment.Left
top.Parent = main

local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 28, 0, 24)
minBtn.Position = UDim2.new(1, -34, 0, 4)
minBtn.Text = "–"
minBtn.TextSize = 18
minBtn.Font = Enum.Font.GothamBold
minBtn.TextColor3 = Color3.new(1, 1, 1)
minBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
minBtn.Parent = main
Instance.new("UICorner", minBtn)

-- LISTA COM ROLAGEM
local list = Instance.new("ScrollingFrame")
list.Position = UDim2.new(0, 6, 0, 34)
list.Size = UDim2.new(1, -12, 1, -40)
list.BackgroundTransparency = 1
list.BorderSizePixel = 0
list.ScrollBarThickness = 5
list.ScrollingDirection = Enum.ScrollingDirection.Y
list.AutomaticCanvasSize = Enum.AutomaticSize.Y
list.CanvasSize = UDim2.new()
list.Parent = main
local layout = Instance.new("UIListLayout", list)
layout.Padding = UDim.new(0, 5)
layout.SortOrder = Enum.SortOrder.LayoutOrder

local order = 0
local function nextOrder() order += 1 return order end

local function section(text)
	local l = Instance.new("TextLabel")
	l.Size = UDim2.new(1, -6, 0, 18)
	l.BackgroundTransparency = 1
	l.Text = text
	l.TextColor3 = Color3.fromRGB(150, 150, 170)
	l.Font = Enum.Font.GothamBold
	l.TextSize = 12
	l.TextXAlignment = Enum.TextXAlignment.Left
	l.LayoutOrder = nextOrder()
	l.Parent = list
end

local function paint(b, on, name)
	b.Text = name .. (on and ": LIGADO" or ": DESLIGADO")
	b.BackgroundColor3 = on and Color3.fromRGB(40, 160, 70) or Color3.fromRGB(170, 45, 45)
end

local function toggle(name, key, onChange)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(1, -6, 0, 32)
	b.Font = Enum.Font.GothamBold
	b.TextSize = 13
	b.TextColor3 = Color3.new(1, 1, 1)
	b.AutoButtonColor = true
	b.LayoutOrder = nextOrder()
	b.Parent = list
	Instance.new("UICorner", b)
	paint(b, state[key], name)
	b.Activated:Connect(function()
		state[key] = not state[key]
		paint(b, state[key], name)
		local ok, err = pcall(onChange, state[key])
		if not ok then warn("[PainelTeste] Erro em " .. name .. ": " .. tostring(err)) end
	end)
	return b
end

local function stepper(label, getText, onMinus, onPlus)
	local row = Instance.new("Frame")
	row.Size = UDim2.new(1, -6, 0, 30)
	row.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
	row.LayoutOrder = nextOrder()
	row.Parent = list
	Instance.new("UICorner", row)
	local t = Instance.new("TextLabel")
	t.Size = UDim2.new(1, -70, 1, 0)
	t.Position = UDim2.new(0, 8, 0, 0)
	t.BackgroundTransparency = 1
	t.TextColor3 = Color3.new(1, 1, 1)
	t.Font = Enum.Font.Gotham
	t.TextSize = 12
	t.TextXAlignment = Enum.TextXAlignment.Left
	t.Parent = row
	local function refresh() t.Text = label .. ": " .. getText() end
	refresh()
	local function mk(txt, x, fn)
		local b = Instance.new("TextButton")
		b.Size = UDim2.new(0, 28, 0, 24)
		b.Position = UDim2.new(1, x, 0.5, -12)
		b.Text = txt
		b.Font = Enum.Font.GothamBold
		b.TextSize = 16
		b.TextColor3 = Color3.new(1, 1, 1)
		b.BackgroundColor3 = Color3.fromRGB(70, 70, 90)
		b.Parent = row
		Instance.new("UICorner", b)
		b.Activated:Connect(function() fn() refresh() end)
	end
	mk("−", -62, onMinus)
	mk("+", -31, onPlus)
end

-- minimizar
local minimized = false
minBtn.Activated:Connect(function()
	minimized = not minimized
	list.Visible = not minimized
	limit.MaxSize = minimized and Vector2.new(230, 32) or Vector2.new(230, 300)
	minBtn.Text = minimized and "+" or "–"
end)

-- arrastar pelo título
do
	local dragging, startPos, startInput
	top.Active = true
	top.InputBegan:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging, startInput, startPos = true, i.Position, main.Position
		end
	end)
	bind(UIS.InputChanged:Connect(function(i)
		if dragging and (i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseMovement) then
			local d = i.Position - startInput
			main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
		end
	end))
	bind(UIS.InputEnded:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = false
		end
	end))
end

-- ================= MOVIMENTO =================
local function applyMovement()
	local h = getHum()
	if not h then return end
	h.WalkSpeed = state.speed and state.speedValue or DEFAULT_SPEED
	h.UseJumpPower = true
	h.JumpPower = state.jump and SUPER_JUMP or DEFAULT_JUMP
end

-- ================= ESP =================
local highlights = {}
local function addESP(p)
	if p == player or not p.Character then return end
	if highlights[p] then highlights[p]:Destroy() end
	local hl = Instance.new("Highlight")
	hl.FillColor = Color3.fromRGB(255, 50, 50)
	hl.OutlineColor = Color3.new(1, 1, 1)
	hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	hl.Adornee = p.Character
	hl.Parent = gui
	highlights[p] = hl
end
local function clearESP()
	for _, hl in pairs(highlights) do hl:Destroy() end
	highlights = {}
end
local function refreshESP()
	clearESP()
	if not state.esp then return end
	for _, p in ipairs(Players:GetPlayers()) do addESP(p) end
end
local function watchPlayer(p)
	bind(p.CharacterAdded:Connect(function()
		task.wait(0.3)
		if state.esp then addESP(p) end
	end))
end
for _, p in ipairs(Players:GetPlayers()) do watchPlayer(p) end
bind(Players.PlayerAdded:Connect(watchPlayer))
bind(Players.PlayerRemoving:Connect(function(p)
	if highlights[p] then highlights[p]:Destroy() highlights[p] = nil end
end))

-- ================= NOCLIP =================
local touched = {}
local function restoreCollisions()
	for part, was in pairs(touched) do
		if part.Parent then part.CanCollide = was end
	end
	touched = {}
end
bind(RunService.Stepped:Connect(function()
	if not state.noclip then return end
	local c = getChar()
	if not c then return end
	for _, d in ipairs(c:GetDescendants()) do
		if d:IsA("BasePart") and d.CanCollide then
			if touched[d] == nil then touched[d] = true end
			d.CanCollide = false
		end
	end
end))

-- ================= SKYWALK =================
local PLATFORM_SIZE = Vector3.new(8, 1, 8)
local SKY_OFFSET, GRID, RADIUS, MAX_PLAT = 50, 8, 2, 200
local platforms, platOrder = {}, {}
local skyConn, skyY, lastX, lastZ
local platFolder = Instance.new("Folder")
platFolder.Name = "SkyWalkPlataformas"
platFolder.Parent = workspace

local function cellOf(pos) return math.floor(pos.X / GRID), math.floor(pos.Z / GRID) end
local function makePlat(x, z)
	local key = x .. ":" .. z
	if platforms[key] then return end
	local p = Instance.new("Part")
	p.Size = PLATFORM_SIZE
	p.Anchored = true
	p.CanCollide = true
	p.Transparency = 1
	p.Position = Vector3.new(x * GRID + GRID / 2, skyY, z * GRID + GRID / 2)
	p.Parent = platFolder
	platforms[key] = p
	table.insert(platOrder, key)
	if #platOrder > MAX_PLAT then
		local k = table.remove(platOrder, 1)
		if platforms[k] then platforms[k]:Destroy() platforms[k] = nil end
	end
end
local function spawnAround(x, z)
	for dx = -RADIUS, RADIUS do
		for dz = -RADIUS, RADIUS do makePlat(x + dx, z + dz) end
	end
end
local function clearPlatforms()
	platFolder:ClearAllChildren()
	platforms, platOrder = {}, {}
end
local function setSky(on)
	if skyConn then skyConn:Disconnect() skyConn = nil end
	clearPlatforms()
	if not on then return end
	local hrp = getHRP()
	if not hrp then return end
	skyY = hrp.Position.Y + SKY_OFFSET
	lastX, lastZ = cellOf(hrp.Position)
	spawnAround(lastX, lastZ)
	hrp.CFrame = CFrame.new(hrp.Position.X, skyY + 5, hrp.Position.Z)
	skyConn = RunService.Heartbeat:Connect(function()
		local r = getHRP()
		if not r then return end
		local x, z = cellOf(r.Position)
		if x ~= lastX or z ~= lastZ then
			spawnAround(x, z)
			lastX, lastZ = x, z
		end
	end)
end

-- ================= AIMBOT (FOV + trava) =================
local PARTS = { "Cabeça", "Torso", "Pé" }
local function partOf(c)
	if state.aimPart == "Cabeça" then return c:FindFirstChild("Head") end
	if state.aimPart == "Torso" then return c:FindFirstChild("UpperTorso") or c:FindFirstChild("Torso") or c:FindFirstChild("HumanoidRootPart") end
	return c:FindFirstChild("RightFoot") or c:FindFirstChild("LeftFoot") or c:FindFirstChild("Right Leg") or c:FindFirstChild("HumanoidRootPart")
end

local circle = Instance.new("Frame")
circle.AnchorPoint = Vector2.new(0.5, 0.5)
circle.Position = UDim2.new(0.5, 0, 0.5, 0)
circle.BackgroundTransparency = 1
circle.Visible = false
circle.Parent = gui
Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)
local stroke = Instance.new("UIStroke", circle)
stroke.Color = Color3.new(1, 1, 1)
stroke.Thickness = 1.5
local function updateCircle() circle.Size = UDim2.new(0, state.fov * 2, 0, state.fov * 2) end
updateCircle()

local rayParams = RaycastParams.new()
rayParams.FilterType = Enum.RaycastFilterType.Exclude

local function isEnemy(p)
	if p == player then return false end
	if player.Team and p.Team and player.Team == p.Team then return false end
	return true
end

local function visible(part, c)
	local origin = camera.CFrame.Position
	rayParams.FilterDescendantsInstances = { getChar(), platFolder }
	local hit = workspace:Raycast(origin, part.Position - origin, rayParams)
	return hit == nil or hit.Instance:IsDescendantOf(c)
end

local function onScreen(part)
	local pos, inView = camera:WorldToViewportPoint(part.Position)
	return inView and pos.Z > 0, Vector2.new(pos.X, pos.Y)
end

local function validTarget(p)
	local c = p and p.Character
	local h = c and c:FindFirstChildOfClass("Humanoid")
	if not h or h.Health <= 0 then return nil end
	local part = partOf(c)
	if not part then return nil end
	local ok = onScreen(part)
	if not ok or not visible(part, c) then return nil end
	return part
end

local locked = nil
local function findTarget()
	local center = camera.ViewportSize / 2
	local best, bestDist = nil, state.fov
	for _, p in ipairs(Players:GetPlayers()) do
		if isEnemy(p) then
			local part = validTarget(p)
			if part then
				local _, sp = onScreen(part)
				local d = (sp - center).Magnitude
				if d < bestDist then best, bestDist = p, d end
			end
		end
	end
	return best
end

RunService:BindToRenderStep("PainelTesteAim", Enum.RenderPriority.Camera.Value + 1, function(dt)
	circle.Visible = state.aim
	if not state.aim then locked = nil return end
	local part = locked and validTarget(locked)
	if not part then
		locked = findTarget()
		part = locked and validTarget(locked)
	end
	if not part then return end
	local desired = CFrame.lookAt(camera.CFrame.Position, part.Position)
	local alpha = 1 - math.exp(-state.smooth * dt)
	camera.CFrame = camera.CFrame:Lerp(desired, alpha)
end)

-- ================= MONTAR PAINEL =================
section("JOGADOR")
toggle("Speed", "speed", applyMovement)
stepper("Velocidade", function() return tostring(state.speedValue) end,
	function() state.speedValue = math.max(16, state.speedValue - 10) applyMovement() end,
	function() state.speedValue = math.min(500, state.speedValue + 10) applyMovement() end)
toggle("Super Pulo", "jump", applyMovement)
toggle("Noclip", "noclip", function(on) if not on then restoreCollisions() end end)
toggle("SkyWalk", "sky", setSky)

section("VISUAL")
toggle("ESP Player", "esp", refreshESP)

section("MIRA")
toggle("Aimbot", "aim", function() locked = nil end)
stepper("Parte", function() return state.aimPart end,
	function()
		local i = table.find(PARTS, state.aimPart)
		state.aimPart = PARTS[(i - 2) % #PARTS + 1]
	end,
	function()
		local i = table.find(PARTS, state.aimPart)
		state.aimPart = PARTS[i % #PARTS + 1]
	end)
stepper("FOV", function() return tostring(state.fov) end,
	function() state.fov = math.max(40, state.fov - 20) updateCircle() end,
	function() state.fov = math.min(400, state.fov + 20) updateCircle() end)
stepper("Trava", function() return tostring(state.smooth) end,
	function() state.smooth = math.max(5, state.smooth - 5) end,
	function() state.smooth = math.min(60, state.smooth + 5) end)

-- ================= RENASCER / LIMPEZA =================
bind(player.CharacterAdded:Connect(function(c)
	c:WaitForChild("Humanoid")
	touched = {}
	applyMovement()
	if state.sky then
		state.sky = false
		setSky(false)
	end
end))

gui.Destroying:Connect(function()
	RunService:UnbindFromRenderStep("PainelTesteAim")
	if skyConn then skyConn:Disconnect() end
	platFolder:Destroy()
	for _, c in ipairs(connections) do c:Disconnect() end
	clearESP()
end)

applyMovement()
print("[PainelTeste] Painel carregado.")
