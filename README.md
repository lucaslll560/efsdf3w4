local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local mouse = player:GetMouse()
local PlayerGui = player:WaitForChild("PlayerGui")

local parts = {}
local selectedPart = nil
local moveArrows = {}
local clickCount = {}
local deleteButton

-- GUI principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PartCreatorGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0,350,0,260)
frame.Position = UDim2.new(0.3,0,0.3,0)
frame.BackgroundColor3 = Color3.fromRGB(40,40,40)
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,40)
title.Position = UDim2.new(0,0,0,0)
title.BackgroundColor3 = Color3.fromRGB(0,170,255)
title.Text = "Part Creator Avançado"
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 22
title.Parent = frame

local function createField(name, y, placeholder)
	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0,120,0,25)
	label.Position = UDim2.new(0,10,0,y)
	label.BackgroundTransparency = 1
	label.Text = name
	label.TextColor3 = Color3.new(1,1,1)
	label.Font = Enum.Font.SourceSans
	label.TextSize = 18
	label.Parent = frame

	local box = Instance.new("TextBox")
	box.Size = UDim2.new(0,200,0,25)
	box.Position = UDim2.new(0,140,0,y)
	box.PlaceholderText = placeholder
	box.BackgroundColor3 = Color3.fromRGB(60,60,60)
	box.TextColor3 = Color3.new(1,1,1)
	box.Font = Enum.Font.SourceSans
	box.TextSize = 18
	box.ClearTextOnFocus = false
	box.Parent = frame
	return box
end

local nameBox = createField("Nome",50,"MinhaPart")
local sizeBox = createField("Tamanho",90,"4,1,2")
local colorBox = createField("Cor",130,"255,255,255")
local anchorBox = createField("Ancorada",170,"true")

-- Botões principais
local createBtn = Instance.new("TextButton")
createBtn.Size = UDim2.new(0,150,0,35)
createBtn.Position = UDim2.new(0,20,0,210)
createBtn.Text = "Criar Part"
createBtn.BackgroundColor3 = Color3.fromRGB(0,170,255)
createBtn.TextColor3 = Color3.new(1,1,1)
createBtn.Font = Enum.Font.SourceSansBold
createBtn.TextSize = 18
createBtn.Parent = frame

local createCarBtn = Instance.new("TextButton")
createCarBtn.Size = UDim2.new(0,150,0,35)
createCarBtn.Position = UDim2.new(0,180,0,210)
createCarBtn.Text = "Criar Part Carro"
createCarBtn.BackgroundColor3 = Color3.fromRGB(0,170,0)
createCarBtn.TextColor3 = Color3.new(1,1,1)
createCarBtn.Font = Enum.Font.SourceSansBold
createCarBtn.TextSize = 18
createCarBtn.Parent = frame

local deleteAllBtn = Instance.new("TextButton")
deleteAllBtn.Size = UDim2.new(0,150,0,35)
deleteAllBtn.Position = UDim2.new(0,100,0,250)
deleteAllBtn.Text = "Deletar Tudo"
deleteAllBtn.BackgroundColor3 = Color3.fromRGB(255,0,0)
deleteAllBtn.TextColor3 = Color3.new(1,1,1)
deleteAllBtn.Font = Enum.Font.SourceSansBold
deleteAllBtn.TextSize = 16
deleteAllBtn.Parent = frame

-- Criar part
local function createPart(car)
	local sizeVals = string.split(sizeBox.Text,",")
	local colorVals = string.split(colorBox.Text,",")
	local anchor = anchorBox.Text:lower()=="true"

	local part = Instance.new("Part")
	part.Name = nameBox.Text~="" and nameBox.Text or (car and "PartCarro" or "NovaPart")
	part.Size = Vector3.new(tonumber(sizeVals[1]) or 4, tonumber(sizeVals[2]) or 1, tonumber(sizeVals[3]) or 2)
	part.Color = Color3.fromRGB(tonumber(colorVals[1]) or 255, tonumber(colorVals[2]) or 255, tonumber(colorVals[3]) or 255)
	part.Anchored = anchor
	part.Position = player.Character and (player.Character.PrimaryPart.Position+Vector3.new(0,5,0)) or Vector3.new(0,5,0)
	part.Parent = workspace
	parts[#parts+1] = part
	clickCount[part] = 0

	-- Se for carro, adiciona Motor6D ou Placeholder para futura movimentação
	if car then
		local seat = Instance.new("VehicleSeat")
		seat.Size = Vector3.new(2,1,2)
		seat.Position = part.Position + Vector3.new(0,2,0)
		seat.Anchored = false
		seat.Parent = workspace
		parts[#parts+1] = seat
	end
end

createBtn.MouseButton1Click:Connect(function() createPart(false) end)
createCarBtn.MouseButton1Click:Connect(function() createPart(true) end)
deleteAllBtn.MouseButton1Click:Connect(function()
	for _,p in pairs(parts) do
		if p and p.Parent then p:Destroy() end
	end
	parts = {}
	selectedPart = nil
	moveArrows = {}
	clickCount = {}
	if modifyGUI then modifyGUI.Visible = false end
	if deleteButton then deleteButton.Visible = false end
end)

-- Aura
local function applyAura(part)
	if not part:FindFirstChild("SelectionBox") then
		local sel = Instance.new("SelectionBox")
		sel.Adornee = part
		sel.LineThickness = 0.05
		sel.Color3 = Color3.fromRGB(0,255,0)
		sel.SurfaceTransparency = 0.5
		sel.Parent = part
	end
end

local function removeAura(part)
	local sel = part:FindFirstChild("SelectionBox")
	if sel then sel:Destroy() end
end

-- Setas
local function createArrows(part)
	for _,arrow in pairs(moveArrows) do arrow:Destroy() end
	moveArrows = {}

	local size = part.Size
	local directions = {
		{Vector3.new(size.X/2 + 1,0,0),"Direita"},
		{Vector3.new(-size.X/2 -1,0,0),"Esquerda"},
		{Vector3.new(0,0,size.Z/2 +1),"Frente"},
		{Vector3.new(0,0,-size.Z/2 -1),"Trás"},
		{Vector3.new(0,size.Y/2 +1,0),"Cima"},
		{Vector3.new(0,-size.Y/2 -1,0),"Baixo"},
	}

	for _,dir in pairs(directions) do
		local arrow = Instance.new("Part")
		arrow.Size = Vector3.new(1,1,1)
		arrow.Anchored = true
		arrow.CanCollide = false
		arrow.Color = Color3.fromRGB(255,255,0)
		arrow.Position = part.Position + dir[1]
		arrow.Name = dir[2]
		arrow.Parent = workspace

		local click = Instance.new("ClickDetector",arrow)
		click.MaxActivationDistance = 50
		click.MouseClick:Connect(function()
			part.Position = part.Position + dir[1].Unit
		end)

		moveArrows[#moveArrows+1] = arrow
	end
end

-- Atualiza setas
spawn(function()
	while true do
		wait(1)
		if selectedPart then
			for i,arrow in pairs(moveArrows) do
				local dir
				if arrow.Name=="Direita" then dir = Vector3.new(selectedPart.Size.X/2 +1,0,0)
				elseif arrow.Name=="Esquerda" then dir = Vector3.new(-selectedPart.Size.X/2 -1,0,0)
				elseif arrow.Name=="Frente" then dir = Vector3.new(0,0,selectedPart.Size.Z/2 +1)
				elseif arrow.Name=="Trás" then dir = Vector3.new(0,0,-selectedPart.Size.Z/2 -1)
				elseif arrow.Name=="Cima" then dir = Vector3.new(0,selectedPart.Size.Y/2 +1,0)
				elseif arrow.Name=="Baixo" then dir = Vector3.new(0,-selectedPart.Size.Y/2 -1,0)
				end
				if dir then arrow.Position = selectedPart.Position + dir end
			end
		end
	end
end)

-- GUI de modificação
modifyGUI = Instance.new("Frame")
modifyGUI.Size = UDim2.new(0,250,0,200)
modifyGUI.Position = UDim2.new(0.5,-125,0.3,0)
modifyGUI.BackgroundColor3 = Color3.fromRGB(60,60,60)
modifyGUI.Visible = false
modifyGUI.Parent = screenGui

local modTitle = Instance.new("TextLabel")
modTitle.Size = UDim2.new(1,0,0,30)
modTitle.BackgroundColor3 = Color3.fromRGB(0,170,255)
modTitle.Text = "Modificar Part"
modTitle.TextColor3 = Color3.new(1,1,1)
modTitle.Font = Enum.Font.SourceSansBold
modTitle.TextSize = 18
modTitle.Parent = modifyGUI

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0,60,0,25)
closeBtn.Position = UDim2.new(1,-65,0,2)
closeBtn.Text = "X"
closeBtn.BackgroundColor3 = Color3.fromRGB(255,0,0)
closeBtn.TextColor3 = Color3.new(1,1,1)
closeBtn.Font = Enum.Font.SourceSansBold
closeBtn.TextSize = 16
closeBtn.Parent = modifyGUI
closeBtn.MouseButton1Click:Connect(function()
	modifyGUI.Visible = false
end)

local function addButton(name, y, func)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0,120,0,30)
	btn.Position = UDim2.new(0,10,y)
	btn.Text = name
	btn.BackgroundColor3 = Color3.fromRGB(0,170,255)
	btn.TextColor3 = Color3.new(1,1,1)
	btn.Font = Enum.Font.SourceSansBold
	btn.TextSize = 16
	btn.Parent = modifyGUI
	btn.MouseButton1Click:Connect(func)
end

addButton("Bloco",40,function() if selectedPart then selectedPart.Shape = Enum.PartType.Block end end)
addButton("Bola",70,function() if selectedPart then selectedPart.Shape = Enum.PartType.Ball end end)
addButton("Cilindro",100,function() if selectedPart then selectedPart.Shape = Enum.PartType.Cylinder end end)
addButton("Deletar",130,function() 
	if selectedPart then
		selectedPart:Destroy()
		modifyGUI.Visible = false
		selectedPart = nil
		for _,arrow in pairs(moveArrows) do arrow:Destroy() end
		moveArrows = {}
	end
end)

-- Botão flutuante distante
deleteButton = Instance.new("TextButton")
deleteButton.Size = UDim2.new(0,80,0,30)
deleteButton.BackgroundColor3 = Color3.fromRGB(255,0,0)
deleteButton.TextColor3 = Color3.new(1,1,1)
deleteButton.Text = "Deletar"
deleteButton.Visible = false
deleteButton.Parent = screenGui
deleteButton.MouseButton1Click:Connect(function()
	if selectedPart then
		selectedPart:Destroy()
		deleteButton.Visible = false
		modifyGUI.Visible = false
		selectedPart = nil
		for _,arrow in pairs(moveArrows) do arrow:Destroy() end
		moveArrows = {}
	end
end)

-- Atualiza posição do botão flutuante mais distante
RunService.RenderStepped:Connect(function()
	if selectedPart and selectedPart.Parent then
		local screenPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(selectedPart.Position + Vector3.new(0, selectedPart.Size.Y + 3,0))
		if onScreen then
			deleteButton.Position = UDim2.new(0, screenPos.X, 0, screenPos.Y)
			deleteButton.Visible = true
		else
			deleteButton.Visible = false
		end
	else
		deleteButton.Visible = false
	end
end)

-- Mouse
RunService.RenderStepped:Connect(function()
	local target = mouse.Target
	for _,p in pairs(parts) do
		if p == target then applyAura(p) else removeAura(p) end
	end
end)

-- Clique na part
mouse.Button1Down:Connect(function()
	local target = mouse.Target
	if not target then return end
	for _,p in pairs(parts) do
		if p == target then
			selectedPart = p
			clickCount[p] = (clickCount[p] or 0) +1

			if clickCount[p] == 1 then
				applyAura(p)
			elseif clickCount[p] == 2 then
				if #moveArrows == 0 then
					createArrows(p)
				else
					for _,arrow in pairs(moveArrows) do arrow:Destroy() end
					moveArrows = {}
				end
			elseif clickCount[p] >= 3 then
				modifyGUI.Visible = true
				clickCount[p] = 0
			end
			break
		end
	end
end)
