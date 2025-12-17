local tool = script.Parent
local event = game.ReplicatedStorage:WaitForChild("RouboFantasma")

tool.Activated:Connect(function()
	event:FireServer()
end)
local event = game.ReplicatedStorage:WaitForChild("RouboFantasma")
local mapa = workspace -- pode trocar por uma pasta específica do mapa

local TEMPO_INVISIVEL = 5

local function pontoAleatorio()
	local partes = mapa:GetDescendants()
	local validos = {}

	for _, obj in ipairs(partes) do
		if obj:IsA("Part") and obj.Anchored then
			table.insert(validos, obj)
		end
	end

	if #validos == 0 then return nil end

	local escolhido = validos[math.random(1, #validos)]
	return escolhido.Position + Vector3.new(0, 5, 0)
end

event.OnServerEvent:Connect(function(player)
	local char = player.Character
	if not char then return end

	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hrp then return end

	-- 🫥 Invisibilidade + despedaçar
	for _, part in ipairs(char:GetChildren()) do
		if part:IsA("BasePart") then
			part.Transparency = 1
			part.CanCollide = false
		end
	end

	-- Teleporte aleatório
	local destino = pontoAleatorio()
	if destino then
		hrp.CFrame = CFrame.new(destino)
	end

	-- ⏳ Tempo invisível
	task.delay(TEMPO_INVISIVEL, function()
		if not char or not char.Parent then return end

		for _, part in ipairs(char:GetChildren()) do
			if part:IsA("BasePart") then
				part.Transparency = 0
				part.CanCollide = true
			end
		end
	end)
end)
local Players = game:GetService("Players")
local localPlayer = Players.LocalPlayer

local function criarESP(character, player)
	if character:FindFirstChild("ESP_Highlight") then return end

	-- 🔦 Highlight
	local highlight = Instance.new("Highlight")
	highlight.Name = "ESP_Highlight"
	highlight.FillTransparency = 0.7
	highlight.OutlineTransparency = 0.2
	highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
	highlight.FillColor = Color3.fromRGB(255, 170, 170)
	highlight.Parent = character

	-- 🏷️ Nome pequeno
	local head = character:WaitForChild("Head")

	local billboard = Instance.new("BillboardGui")
	billboard.Name = "ESP_Name"
	billboard.Size = UDim2.new(0, 100, 0, 20)
	billboard.StudsOffset = Vector3.new(0, 2.5, 0)
	billboard.AlwaysOnTop = true
	billboard.Parent = head

	local text = Instance.new("TextLabel")
	text.Size = UDim2.new(1, 0, 1, 0)
	text.BackgroundTransparency = 1
	text.Text = player.Name
	text.TextScaled = true
	text.Font = Enum.Font.Gotham
	text.TextColor3 = Color3.fromRGB(255, 255, 255)
	text.TextStrokeTransparency = 0.6
	text.Parent = billboard
end

local function onPlayerAdded(player)
	if player == localPlayer then return end

	player.CharacterAdded:Connect(function(character)
		task.wait(0.5)
		criarESP(character, player)
	end)
end

-- Jogadores já no servidor
for _, player in ipairs(Players:GetPlayers()) do
	onPlayerAdded(player)
	if player.Character then
		criarESP(player.Character, player)
	end
end

Players.PlayerAdded:Connect(onPlayerAdded)
