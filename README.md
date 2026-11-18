--==============================================================
--  JEFIN MENU — TELA DE LICENÇA (visual + validação real)
--  key fixa na lista + validade de 7 dias a partir do PRIMEIRO
--  uso, contado de verdade no Firebase (não reseta por PC).
--==============================================================
local Players          = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local HttpService      = game:GetService("HttpService")
local LocalPlayer      = Players.LocalPlayer

--==============================================================
--  CONFIGURAÇÃO DA KEY — edite aqui
--==============================================================
-- troque pela URL do seu Worker (ex: "https://jefin-menu.SEUNOME.workers.dev")
local WORKER_URL = "https://jefin-menu.jeffmarttjr.workers.dev"

-- não tem mais lista de keys nem Firebase aqui de propósito: quem decide se a
-- key é válida, se já expirou e o que libera é o Worker — o jogador (ou quem
-- comprou o script pirateado) não consegue ler a lista de keys nem o menu
-- real só lendo esse arquivo.
local function pedirHttp()
	return request or http_request or (type(syn) == "table" and syn.request) or nil
end

local function pedirMenu(key)
	local req = pedirHttp()
	if not req then
		return nil, "executor sem request/http_request"
	end
	if WORKER_URL == "https://jefin-menu.jeffmarttjr.workers.dev/" or WORKER_URL == "" then
		return nil, "WORKER_URL não configurada (veja o topo do script)"
	end

	local url = WORKER_URL .. "?key=" .. HttpService:UrlEncode(key)
	local ok, res = pcall(req, { Url = url, Method = "GET" })
	if not ok or not res then
		return nil, "falha ao falar com o servidor (sem internet?)"
	end
	if res.StatusCode == 200 then
		return res.Body
	end
	-- o Worker manda o motivo em texto simples no corpo, sem ser código Lua
	local motivo = res.Body or ""
	motivo = motivo:match('"%[Jefin Key%]%s*(.-)"') or motivo
	return nil, (motivo ~= "" and motivo or "key recusada")
end

--==============================================================
--  MARCA
--==============================================================
local MARCA = {
	Nome    = "JEFIN MENU",
	Slogan  = "acesso por key — 7 dias desde o 1º uso",
	Versao  = "v1.0",
	Discord = "discord.gg/SEU-SERVIDOR", -- troque pelo seu
}

--==============================================================
--  TEMA
--==============================================================
local T = {
	Fundo       = Color3.fromRGB(15, 15, 18),
	Barra       = Color3.fromRGB(19, 19, 23),
	Campo       = Color3.fromRGB(23, 23, 28),
	CampoFoco   = Color3.fromRGB(27, 27, 33),
	Borda       = Color3.fromRGB(38, 38, 45),
	BordaForte  = Color3.fromRGB(58, 58, 68),
	Accent      = Color3.fromRGB(226, 54, 50),
	AccentEsc   = Color3.fromRGB(150, 30, 28),
	AccentSuave = Color3.fromRGB(255, 116, 110),
	Texto       = Color3.fromRGB(238, 238, 243),
	TextoMedio  = Color3.fromRGB(158, 158, 170),
	TextoFraco  = Color3.fromRGB(104, 104, 116),
	Ok          = Color3.fromRGB(72, 199, 116),
	Erro        = Color3.fromRGB(226, 84, 84),
	Alerta      = Color3.fromRGB(226, 176, 72),
}
local FONTE  = Enum.Font.Gotham
local F_BOLD = Enum.Font.GothamBold
local F_MED  = Enum.Font.GothamMedium
local F_MONO = Enum.Font.Code

--==============================================================
--  HELPERS
--==============================================================
local function new(class, props, children)
	local inst = Instance.new(class)
	local parent = nil
	if props then
		for k, v in pairs(props) do
			if k == "Parent" then parent = v else inst[k] = v end
		end
	end
	if children then
		for _, c in ipairs(children) do c.Parent = inst end
	end
	if parent then inst.Parent = parent end
	return inst
end
local function canto(inst, raio)
	return new("UICorner", { CornerRadius = UDim.new(0, raio), Parent = inst })
end
local function traco(inst, cor, esp, gradiente)
	local s = new("UIStroke", {
		Color = cor or T.Borda,
		Thickness = esp or 1,
		ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
		Parent = inst,
	})
	if gradiente then
		new("UIGradient", { Color = gradiente, Rotation = 90, Parent = s })
	end
	return s
end
local function tween(inst, tempo, props, estilo, dir)
	local t = TweenService:Create(
		inst,
		TweenInfo.new(tempo, estilo or Enum.EasingStyle.Quad, dir or Enum.EasingDirection.Out),
		props
	)
	t:Play()
	return t
end
local function getGuiParent()
	if typeof(gethui) == "function" then
		local ok, h = pcall(gethui)
		if ok and h then return h end
	end
	local ok, cg = pcall(function() return game:GetService("CoreGui") end)
	if ok and cg then
		local teste = Instance.new("Folder")
		local escreveu = pcall(function() teste.Parent = cg end)
		teste:Destroy()
		if escreveu then return cg end
	end
	return LocalPlayer:WaitForChild("PlayerGui")
end
local GUI_PARENT = getGuiParent()

--==============================================================
--  JANELA
--==============================================================
local NOME = "JefinKey"
local antigo = GUI_PARENT:FindFirstChild(NOME)
if antigo then antigo:Destroy() end

local LARG, ALT = 440, 372
local BARRA_H = 40
local gui = new("ScreenGui", {
	Name = NOME,
	ResetOnSpawn = false,
	ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
	Parent = GUI_PARENT,
})

local sombra = new("Frame", {
	Size = UDim2.new(0, LARG + 14, 0, ALT + 14),
	Position = UDim2.new(0.5, -(LARG + 14) / 2, 0.5, -(ALT + 14) / 2 + 5),
	BackgroundColor3 = Color3.new(0, 0, 0),
	BackgroundTransparency = 0.5,
	BorderSizePixel = 0,
	ZIndex = 0,
	Parent = gui,
})
canto(sombra, 20)

local card = new("Frame", {
	Name = "Card",
	Size = UDim2.new(0, LARG, 0, ALT),
	Position = UDim2.new(0.5, -LARG / 2, 0.5, -ALT / 2),
	BackgroundColor3 = T.Fundo,
	BorderSizePixel = 0,
	ClipsDescendants = true,
	Active = true,
	ZIndex = 1,
	Parent = gui,
})
canto(card, 14)
traco(card, nil, 1, ColorSequence.new({
	ColorSequenceKeypoint.new(0, Color3.fromRGB(72, 34, 34)),
	ColorSequenceKeypoint.new(0.5, T.Borda),
	ColorSequenceKeypoint.new(1, Color3.fromRGB(28, 28, 34)),
}))

--==============================================================
--  DRAG
--==============================================================
local function arrastavel(frame, alca)
	alca = alca or frame
	local ativo, inicio, base = false, nil, nil
	alca.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
			or input.UserInputType == Enum.UserInputType.Touch then
			ativo = true
			inicio = input.Position
			base = frame.Position
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if not ativo then return end
		if input.UserInputType == Enum.UserInputType.MouseMovement
			or input.UserInputType == Enum.UserInputType.Touch then
			local d = input.Position - inicio
			local nova = UDim2.new(base.X.Scale, base.X.Offset + d.X, base.Y.Scale, base.Y.Offset + d.Y)
			frame.Position = nova
			sombra.Position = UDim2.new(base.X.Scale, base.X.Offset + d.X - 7, base.Y.Scale, base.Y.Offset + d.Y + 5)
		end
	end)
	UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
			or input.UserInputType == Enum.UserInputType.Touch then
			ativo = false
		end
	end)
end

--==============================================================
--  BARRA SUPERIOR
--==============================================================
local barra = new("Frame", {
	Name = "Barra",
	Size = UDim2.new(1, 0, 0, BARRA_H),
	BackgroundColor3 = T.Barra,
	BorderSizePixel = 0,
	ZIndex = 2,
	Parent = card,
})
local faixa = new("Frame", {
	Name = "Faixa",
	Size = UDim2.new(1, 0, 0, 2),
	BackgroundColor3 = T.Accent,
	BorderSizePixel = 0,
	ZIndex = 4,
	Parent = card,
})
local faixaGrad = new("UIGradient", {
	Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, T.Accent),
		ColorSequenceKeypoint.new(0.6, T.AccentEsc),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 30, 36)),
	}),
	Parent = faixa,
})
new("TextLabel", {
	Size = UDim2.new(0, 300, 1, 0),
	Position = UDim2.new(0, 50, 0, 0),
	BackgroundTransparency = 1,
	Text = MARCA.Nome .. "  " .. MARCA.Versao,
	Font = F_BOLD,
	TextSize = 12,
	TextColor3 = T.TextoMedio,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 3,
	Parent = barra,
})
new("TextLabel", {
	Size = UDim2.new(0, 150, 1, 0),
	Position = UDim2.new(1, -186, 0, 0),
	BackgroundTransparency = 1,
	Text = "licenciamento",
	Font = FONTE,
	TextSize = 11,
	TextColor3 = T.TextoFraco,
	TextXAlignment = Enum.TextXAlignment.Right,
	ZIndex = 3,
	Parent = barra,
})
local marcas = new("Frame", {
	Name = "Marcas",
	Size = UDim2.new(0, 26, 0, 3),
	Position = UDim2.new(0, 18, 0.5, -1.5),
	BackgroundColor3 = Color3.fromRGB(46, 46, 54),
	BorderSizePixel = 0,
	ZIndex = 3,
	Parent = barra,
})
canto(marcas, 2)
for i = 1, 3 do
	new("Frame", {
		Size = UDim2.new(0, 3, 0, 3),
		Position = UDim2.new(0, (i - 1) * 5, 0, 0),
		BackgroundColor3 = Color3.fromRGB(70, 70, 80),
		BorderSizePixel = 0,
		ZIndex = 4,
		Parent = marcas,
	})
end
local btnFechar = new("TextButton", {
	Size = UDim2.new(0, 28, 0, 28),
	Position = UDim2.new(1, -38, 0.5, -14),
	BackgroundColor3 = Color3.fromRGB(30, 30, 36),
	Text = "X",
	Font = F_BOLD,
	TextSize = 13,
	TextColor3 = T.TextoFraco,
	BorderSizePixel = 0,
	AutoButtonColor = false,
	ZIndex = 3,
	Parent = barra,
})
canto(btnFechar, 7)
btnFechar.MouseEnter:Connect(function()
	tween(btnFechar, 0.15, { BackgroundColor3 = T.Accent, TextColor3 = Color3.new(1, 1, 1) })
end)
btnFechar.MouseLeave:Connect(function()
	tween(btnFechar, 0.15, { BackgroundColor3 = Color3.fromRGB(30, 30, 36), TextColor3 = T.TextoFraco })
end)
arrastavel(card, barra)

--==============================================================
--  LOGO + TÍTULOS
--==============================================================
local logoTile = new("Frame", {
	Name = "Logo",
	Size = UDim2.new(0, 46, 0, 46),
	Position = UDim2.new(0.5, -23, 0, 56),
	BackgroundColor3 = T.Accent,
	BorderSizePixel = 0,
	ZIndex = 3,
	Parent = card,
})
canto(logoTile, 13)
new("UIGradient", {
	Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, Color3.fromRGB(240, 72, 68)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(132, 26, 24)),
	}),
	Rotation = 90,
	Parent = logoTile,
})
traco(logoTile, Color3.fromRGB(255, 120, 114), 1)
new("TextLabel", {
	Size = UDim2.new(1, 0, 1, 0),
	BackgroundTransparency = 1,
	Text = "J",
	Font = F_BOLD,
	TextSize = 24,
	TextColor3 = Color3.new(1, 1, 1),
	ZIndex = 4,
	Parent = logoTile,
})
new("TextLabel", {
	Size = UDim2.new(1, -64, 0, 22),
	Position = UDim2.new(0, 32, 0, 112),
	BackgroundTransparency = 1,
	Text = MARCA.Nome,
	Font = F_BOLD,
	TextSize = 19,
	TextColor3 = T.Texto,
	ZIndex = 3,
	Parent = card,
})
new("TextLabel", {
	Size = UDim2.new(1, -64, 0, 15),
	Position = UDim2.new(0, 32, 0, 135),
	BackgroundTransparency = 1,
	Text = MARCA.Slogan,
	Font = FONTE,
	TextSize = 11,
	TextColor3 = T.TextoFraco,
	ZIndex = 3,
	Parent = card,
})
new("Frame", {
	Size = UDim2.new(1, -64, 0, 1),
	Position = UDim2.new(0, 32, 0, 166),
	BackgroundColor3 = T.Borda,
	BorderSizePixel = 0,
	ZIndex = 3,
	Parent = card,
})

--==============================================================
--  CAMPO DE KEY
--==============================================================
new("TextLabel", {
	Name = "Rotulo",
	Size = UDim2.new(0, 260, 0, 12),
	Position = UDim2.new(0, 32, 0, 180),
	BackgroundTransparency = 1,
	Text = "CHAVE DE ACESSO",
	Font = F_BOLD,
	TextSize = 10,
	TextColor3 = T.TextoFraco,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 3,
	Parent = card,
})
local contador = new("TextLabel", {
	Name = "Contador",
	Size = UDim2.new(0, 90, 0, 12),
	Position = UDim2.new(1, -122, 0, 180),
	BackgroundTransparency = 1,
	Text = "0/20",
	Font = F_MONO,
	TextSize = 10,
	TextColor3 = T.TextoFraco,
	TextXAlignment = Enum.TextXAlignment.Right,
	ZIndex = 3,
	Parent = card,
})
local campoBox = new("Frame", {
	Name = "Campo",
	Size = UDim2.new(1, -64, 0, 46),
	Position = UDim2.new(0, 32, 0, 196),
	BackgroundColor3 = T.Campo,
	BorderSizePixel = 0,
	ZIndex = 3,
	Parent = card,
})
canto(campoBox, 10)
local campoStroke = traco(campoBox, T.Borda, 1)
local campo = new("TextBox", {
	Name = "Input",
	Size = UDim2.new(1, -28, 1, 0),
	Position = UDim2.new(0, 14, 0, 0),
	BackgroundTransparency = 1,
	Text = "",
	PlaceholderText = "XXXXX-XXXXX-XXXXX-XXXXX",
	PlaceholderColor3 = Color3.fromRGB(72, 72, 84),
	Font = F_MONO,
	TextSize = 16,
	TextColor3 = T.Texto,
	TextXAlignment = Enum.TextXAlignment.Center,
	ClearTextOnFocus = false,
	MultiLine = false,
	ZIndex = 4,
	Parent = campoBox,
})
campo.Focused:Connect(function()
	tween(campoBox, 0.15, { BackgroundColor3 = T.CampoFoco })
	tween(campoStroke, 0.15, { Color = T.Accent })
end)
campo.FocusLost:Connect(function()
	tween(campoBox, 0.15, { BackgroundColor3 = T.Campo })
	tween(campoStroke, 0.15, { Color = T.Borda })
end)

-- formatação visual: MAIÚSCULO + hífen a cada 5 caracteres
local formatando = false
local function formatarKey(txt)
	local limpo = txt:upper():gsub("[^A-Z0-9]", "")
	limpo = limpo:sub(1, 20)
	local grupos = {}
	for i = 1, #limpo, 5 do
		table.insert(grupos, limpo:sub(i, i + 4))
	end
	return table.concat(grupos, "-")
end
campo:GetPropertyChangedSignal("Text"):Connect(function()
	if formatando then return end
	formatando = true
	local fmt = formatarKey(campo.Text)
	campo.Text = fmt
	campo.CursorPosition = #fmt + 1
	local limpo = fmt:gsub("-", "")
	contador.Text = #limpo .. "/20"
	contador.TextColor3 = (#limpo == 20) and T.Ok or T.TextoFraco
	formatando = false
end)

--==============================================================
--  STATUS
--==============================================================
local statusBox = new("Frame", {
	Name = "Status",
	Size = UDim2.new(0, 380, 0, 16),
	Position = UDim2.new(0, 32, 0, 252),
	BackgroundTransparency = 1,
	ZIndex = 3,
	Parent = card,
})
local statusDot = new("Frame", {
	Size = UDim2.new(0, 6, 0, 6),
	Position = UDim2.new(0, 0, 0.5, -3),
	BackgroundColor3 = T.TextoFraco,
	BorderSizePixel = 0,
	ZIndex = 4,
	Parent = statusBox,
})
canto(statusDot, 3)
local statusTexto = new("TextLabel", {
	Size = UDim2.new(1, -14, 1, 0),
	Position = UDim2.new(0, 14, 0, 0),
	BackgroundTransparency = 1,
	Text = "aguardando chave de acesso",
	Font = FONTE,
	TextSize = 11,
	TextColor3 = T.TextoFraco,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 4,
	Parent = statusBox,
})
local CORES_STATUS = { ok = T.Ok, erro = T.Erro, alerta = T.Alerta, neutro = T.TextoFraco }
local function definirStatus(tipo, texto)
	local cor = CORES_STATUS[tipo] or T.TextoFraco
	statusDot.BackgroundColor3 = cor
	statusTexto.Text = texto
	statusTexto.TextColor3 = cor
end

--==============================================================
--  BOTÕES
--==============================================================
local btnColar = new("TextButton", {
	Name = "Colar",
	Size = UDim2.new(0, 100, 0, 42),
	Position = UDim2.new(0, 32, 0, 278),
	BackgroundColor3 = Color3.fromRGB(28, 28, 34),
	Text = "COLAR",
	Font = F_BOLD,
	TextSize = 12,
	TextColor3 = T.TextoMedio,
	BorderSizePixel = 0,
	AutoButtonColor = false,
	ZIndex = 3,
	Parent = card,
})
canto(btnColar, 9)
local colarStroke = traco(btnColar, T.Borda, 1)
btnColar.MouseEnter:Connect(function()
	tween(btnColar, 0.15, { BackgroundColor3 = Color3.fromRGB(38, 38, 46), TextColor3 = T.Texto })
	tween(colarStroke, 0.15, { Color = T.BordaForte })
end)
btnColar.MouseLeave:Connect(function()
	tween(btnColar, 0.15, { BackgroundColor3 = Color3.fromRGB(28, 28, 34), TextColor3 = T.TextoMedio })
	tween(colarStroke, 0.15, { Color = T.Borda })
end)
btnColar.MouseButton1Click:Connect(function()
	tween(btnColar, 0.08, { BackgroundColor3 = Color3.fromRGB(46, 46, 56) })
	task.delay(0.12, function()
		tween(btnColar, 0.2, { BackgroundColor3 = Color3.fromRGB(28, 28, 34) })
	end)
	if typeof(getclipboard) ~= "function" then
		definirStatus("alerta", "executor sem getclipboard")
		return
	end
	local ok, txt = pcall(getclipboard)
	if ok and type(txt) == "string" and #txt > 0 then
		campo.Text = txt -- o listener de Text já formata sozinho
	else
		definirStatus("alerta", "área de transferência vazia")
	end
end)

local btnAtivar = new("TextButton", {
	Name = "Ativar",
	Size = UDim2.new(0, 276, 0, 42),
	Position = UDim2.new(0, 144, 0, 278),
	BackgroundColor3 = T.Accent,
	Text = "ATIVAR",
	Font = F_BOLD,
	TextSize = 13,
	TextColor3 = Color3.new(1, 1, 1),
	BorderSizePixel = 0,
	AutoButtonColor = false,
	ClipsDescendants = true,
	ZIndex = 3,
	Parent = card,
})
canto(btnAtivar, 9)
new("UIGradient", {
	Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, Color3.fromRGB(240, 70, 66)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(168, 32, 30)),
	}),
	Rotation = 0,
	Parent = btnAtivar,
})
local progresso = new("Frame", {
	Name = "Progresso",
	Size = UDim2.new(0, 0, 0, 3),
	Position = UDim2.new(0, 0, 1, -3),
	BackgroundColor3 = Color3.new(1, 1, 1),
	BackgroundTransparency = 0.35,
	BorderSizePixel = 0,
	ZIndex = 5,
	Parent = btnAtivar,
})

local carregando = false
local function fimDoLoading()
	carregando = false
	if not btnAtivar.Parent then return end
	btnAtivar.Text = "ATIVAR"
	tween(btnAtivar, 0.2, { BackgroundColor3 = T.Accent })
	tween(progresso, 0.15, { BackgroundTransparency = 1 })
end
btnAtivar.MouseEnter:Connect(function()
	if carregando then return end
	tween(btnAtivar, 0.15, { BackgroundColor3 = T.AccentSuave })
end)
btnAtivar.MouseLeave:Connect(function()
	if carregando then return end
	tween(btnAtivar, 0.15, { BackgroundColor3 = T.Accent })
end)

--------------------------------------------------------------
--  validação de verdade
--------------------------------------------------------------
local function ativarClique()
	if carregando then return end

	local key = campo.Text
	if key == "" then
		definirStatus("erro", "digite uma key")
		return
	end

	carregando = true
	btnAtivar.Text = "VALIDANDO..."
	tween(btnAtivar, 0.15, { BackgroundColor3 = T.AccentEsc })
	progresso.Size = UDim2.new(0, 0, 0, 3)
	tween(progresso, 0.15, { BackgroundTransparency = 0.35 })
	tween(progresso, 1.5, { Size = UDim2.new(1, 0, 0, 3) }, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
	definirStatus("alerta", "checando key...")

	task.spawn(function()
		local codigoDoMenu, erro = pedirMenu(key)
		if not codigoDoMenu then
			definirStatus("erro", erro or "erro desconhecido")
			fimDoLoading()
			return
		end

		definirStatus("ok", "licença ativa — carregando...")
		fimDoLoading()
		task.wait(0.3)
		pcall(function()
			card.Visible = false
			sombra.Visible = false
		end)
		-- o código do menu só existe aqui, no retorno do Worker — nunca fica
		-- salvo neste arquivo nem em nenhum link público fixo.
		local okExec, errExec = pcall(function()
			loadstring(codigoDoMenu)()
		end)
		if not okExec then
			warn("[Jefin Key] erro ao carregar o menu: " .. tostring(errExec))
		end
	end)
end

btnAtivar.MouseButton1Click:Connect(ativarClique)
campo.FocusLost:Connect(function(enterPressed)
	if enterPressed then ativarClique() end
end)

--==============================================================
--  LINKS + RODAPÉ
--==============================================================
new("TextLabel", {
	Size = UDim2.new(0, 200, 0, 14),
	Position = UDim2.new(0, 32, 0, 330),
	BackgroundTransparency = 1,
	Text = "não tem uma chave?",
	Font = FONTE,
	TextSize = 11,
	TextColor3 = T.TextoFraco,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 3,
	Parent = card,
})
local linkDiscord = new("TextButton", {
	Size = UDim2.new(0, 200, 0, 14),
	Position = UDim2.new(0, 132, 0, 330),
	BackgroundTransparency = 1,
	Text = MARCA.Discord,
	Font = F_MED,
	TextSize = 11,
	TextColor3 = T.AccentSuave,
	TextXAlignment = Enum.TextXAlignment.Left,
	AutoButtonColor = false,
	ZIndex = 3,
	Parent = card,
})
linkDiscord.MouseEnter:Connect(function() tween(linkDiscord, 0.15, { TextColor3 = Color3.new(1, 1, 1) }) end)
linkDiscord.MouseLeave:Connect(function() tween(linkDiscord, 0.15, { TextColor3 = T.AccentSuave }) end)
linkDiscord.MouseButton1Click:Connect(function()
	if typeof(setclipboard) == "function" then
		pcall(setclipboard, MARCA.Discord)
		definirStatus("alerta", "link do discord copiado")
	end
end)

local rodape = new("Frame", {
	Size = UDim2.new(1, 0, 0, 26),
	Position = UDim2.new(0, 0, 1, -26),
	BackgroundColor3 = T.Barra,
	BorderSizePixel = 0,
	ZIndex = 3,
	Parent = card,
})
new("Frame", {
	Size = UDim2.new(1, 0, 0, 1),
	BackgroundColor3 = T.Borda,
	BorderSizePixel = 0,
	ZIndex = 4,
	Parent = rodape,
})
new("TextLabel", {
	Size = UDim2.new(0, 260, 1, 0),
	Position = UDim2.new(0, 32, 0, 0),
	BackgroundTransparency = 1,
	Text = "validade  7 dias desde o 1º uso",
	Font = F_MONO,
	TextSize = 10,
	TextColor3 = Color3.fromRGB(80, 80, 92),
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 4,
	Parent = rodape,
})
new("TextLabel", {
	Size = UDim2.new(0, 120, 1, 0),
	Position = UDim2.new(1, -152, 0, 0),
	BackgroundTransparency = 1,
	Text = "conexão segura",
	Font = FONTE,
	TextSize = 10,
	TextColor3 = Color3.fromRGB(80, 80, 92),
	TextXAlignment = Enum.TextXAlignment.Right,
	ZIndex = 4,
	Parent = rodape,
})

--==============================================================
--  SHEEN — brilho percorrendo a faixa superior
--==============================================================
task.spawn(function()
	local pos = -1
	while gui.Parent do
		task.wait(0.03)
		pos += 0.012
		if pos > 2 then pos = -1 end
		if faixaGrad and faixaGrad.Parent then
			faixaGrad.Offset = Vector2.new(pos, 0)
		end
	end
end)

--==============================================================
--  FECHAR / REABRIR
--==============================================================
local reabrir = new("TextButton", {
	Name = "Reabrir",
	Size = UDim2.new(0, 46, 0, 46),
	Position = UDim2.new(0, 20, 0.5, -23),
	BackgroundColor3 = T.Accent,
	Text = "J",
	Font = F_BOLD,
	TextSize = 20,
	TextColor3 = Color3.new(1, 1, 1),
	BorderSizePixel = 0,
	AutoButtonColor = false,
	Visible = false,
	ZIndex = 10,
	Parent = gui,
})
canto(reabrir, 14)
traco(reabrir, Color3.fromRGB(255, 120, 114), 1)
arrastavel(reabrir)
reabrir.MouseEnter:Connect(function() tween(reabrir, 0.15, { BackgroundColor3 = T.AccentSuave }) end)
reabrir.MouseLeave:Connect(function() tween(reabrir, 0.15, { BackgroundColor3 = T.Accent }) end)
reabrir.MouseButton1Click:Connect(function()
	reabrir.Visible = false
	card.Visible = true
	sombra.Visible = true
end)
btnFechar.MouseButton1Click:Connect(function()
	card.Visible = false
	sombra.Visible = false
	reabrir.Visible = true
end)

--==============================================================
--  EXPOSIÇÃO (opcional) — pra controlar o status de fora
--==============================================================
_G.JefinKey = {
	DefinirStatus = definirStatus,
	Loading = function(estado)
		if estado then
			if carregando then return end
			carregando = true
			btnAtivar.Text = "VALIDANDO..."
			progresso.Size = UDim2.new(0, 0, 0, 3)
			tween(progresso, 1.5, { Size = UDim2.new(1, 0, 0, 3) })
		else
			fimDoLoading()
		end
	end,
	Texto = function(t) campo.Text = t end,
	Fechar = function()
		card.Visible = false
		sombra.Visible = false
		reabrir.Visible = true
	end,
}

print("[Jefin Key] tela de licença carregada.")
