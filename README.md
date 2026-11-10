--[[
    ╔═══════════════════════════════════════════╗
    ║        ZyaMe Library  ✦  v1.0             ║
    ║        Modern Roblox UI Library           ║
    ║        Author: mangga_enak9999            ║
    ╚═══════════════════════════════════════════╝
    
    Usage:
        local ZyaMe = loadstring(game:HttpGet("URL_HERE"))()
        -- atau require(path.To.ZyaMe)
        
        local Win = ZyaMe:Window({ Title = "My Hub", Width = 600 })
        local Tab = Win:Tab("Main", "🏠")
        Tab:Section("Settings")
        Tab:Toggle("ESP", "Wallhack", false, function(v) print(v) end)
        Tab:Slider("Speed", 16, 200, 16, function(v) print(v) end)
        Tab:Button("Click", "→", function() print("clicked") end)
        Tab:Input("Name", "Enter...", function(txt) print(txt) end)
        
        local Floats = ZyaMe:Floats()
        local Btn, entry = Floats:Create({
            Name = "MyBtn", Text = "CLICK",
            Size = UDim2.new(0, 60, 0, 60),
            Position = UDim2.new(0, 20, 0, 20),
            Callback = function() print("float click") end
        })
        Floats:ToggleMode(true)   -- tampilkan semua float
        Floats:SetRainbow(true)
]]

local ZyaMe = {}

--// ============================================================
--// SERVICES
--// ============================================================
local UIS = game:GetService("UserInputService")
local TS = game:GetService("TweenService")
local RS = game:GetService("RunService")
local CG = game:GetService("CoreGui")
local SG = game:GetService("StarterGui")
local Players = game:GetService("Players")
local lp = Players.LocalPlayer

--// ============================================================
--// DESIGN TOKENS
--// ============================================================
ZyaMe.Design = {
    Bg        = Color3.fromRGB(15, 15, 20),
    Surface   = Color3.fromRGB(24, 24, 32),
    Surface2  = Color3.fromRGB(32, 32, 42),
    Border    = Color3.fromRGB(48, 48, 62),
    BorderLit = Color3.fromRGB(70, 70, 90),
    Text      = Color3.fromRGB(238, 238, 245),
    TextDim   = Color3.fromRGB(145, 145, 165),
    TextMuted = Color3.fromRGB(90, 90, 110),
    Accent    = Color3.fromRGB(139, 92, 246),
    Accent2   = Color3.fromRGB(217, 70, 239),
    AccentLt  = Color3.fromRGB(167, 139, 250),
    Success   = Color3.fromRGB(52, 211, 153),
    Danger    = Color3.fromRGB(248, 113, 113),
    Warning   = Color3.fromRGB(251, 191, 36),
    Round     = UDim.new(0, 12),
    RoundSm   = UDim.new(0, 8),
    RoundFull = UDim.new(1, 0),
    Font      = Enum.Font.Gotham,
    FontBold  = Enum.Font.GothamBold,
    FontMed   = Enum.Font.GothamMedium,
}

local D = ZyaMe.Design

--// ============================================================
--// UTILS
--// ============================================================
local function tween(obj, time, props, style)
    local t = TS:Create(obj, TweenInfo.new(time or 0.2, style or Enum.EasingStyle.Quart, Enum.EasingDirection.Out), props)
    t:Play()
    return t
end

local function corner(obj, radius)
    local c = Instance.new("UICorner")
    c.CornerRadius = radius or D.Round
    c.Parent = obj
    return c
end

local function stroke(obj, color, thickness)
    local s = Instance.new("UIStroke")
    s.Color = color or D.Border
    s.Thickness = thickness or 1
    s.Parent = obj
    return s
end

local function gradient(obj, c1, c2, rot)
    local g = Instance.new("UIGradient")
    g.Color = ColorSequence.new(c1, c2)
    g.Rotation = rot or 90
    g.Parent = obj
    return g
end

local function notify(title, text, duration)
    pcall(function()
        SG:SetCore("SendNotification", {Title = title, Text = text, Duration = duration or 3})
    end)
end

local function HSLtoRGB(h, s, l)
    local function h2r(p, q, t)
        if t < 0 then t = t + 1 end
        if t > 1 then t = t - 1 end
        if t < 1/6 then return p + (q - p) * 6 * t end
        if t < 1/2 then return q end
        if t < 2/3 then return p + (q - p) * (2/3 - t) * 6 end
        return p
    end
    local r, g, b
    if s == 0 then r, g, b = l, l, l
    else
        local q = l < 0.5 and l * (1 + s) or l + s - l * s
        local p = 2 * l - q
        r = h2r(p, q, h + 1/3); g = h2r(p, q, h); b = h2r(p, q, h - 1/3)
    end
    return Color3.new(r, g, b)
end

ZyaMe.Utils = {
    Tween = tween, Corner = corner, Stroke = stroke, Gradient = gradient,
    Notify = notify, HSLtoRGB = HSLtoRGB,
}

--// ============================================================
--// COMPONENTS
--// ============================================================
local Components = {}

function Components.Section(tab, text)
    local holder = Instance.new("Frame")
    holder.Size = UDim2.new(1, 0, 0, 22)
    holder.BackgroundTransparency = 1
    holder.Parent = tab.Page

    local bar = Instance.new("Frame")
    bar.Size = UDim2.new(0, 3, 0, 12)
    bar.Position = UDim2.new(0, 0, 0.5, -6)
    bar.BackgroundColor3 = D.Accent
    bar.Parent = holder
    corner(bar, D.RoundFull)
    gradient(bar, D.Accent, D.Accent2, 90)

    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, -12, 1, 0)
    lbl.Position = UDim2.new(0, 12, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = string.upper(text)
    lbl.TextColor3 = D.AccentLt
    lbl.Font = D.FontBold
    lbl.TextSize = 10
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = holder
    return holder
end

function Components.Toggle(tab, name, desc, default, callback)
    local Card = Instance.new("Frame")
    Card.Size = UDim2.new(1, 0, 0, desc and 52 or 44)
    Card.BackgroundColor3 = D.Surface
    Card.BorderSizePixel = 0
    Card.Parent = tab.Page
    corner(Card, D.RoundSm)
    local cs = stroke(Card, D.Border, 1)
    cs.Transparency = 0.5

    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, -80, 0, desc and 22 or 44)
    lbl.Position = UDim2.new(0, 14, 0, desc and 6 or 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = name
    lbl.TextColor3 = D.Text
    lbl.Font = D.FontMed
    lbl.TextSize = 13
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = Card

    if desc then
        local d = Instance.new("TextLabel")
        d.Size = UDim2.new(1, -80, 0, 16)
        d.Position = UDim2.new(0, 14, 0, 28)
        d.BackgroundTransparency = 1
        d.Text = desc
        d.TextColor3 = D.TextDim
        d.Font = D.Font
        d.TextSize = 10
        d.TextXAlignment = Enum.TextXAlignment.Left
        d.Parent = Card
    end

    local Switch = Instance.new("Frame")
    Switch.Size = UDim2.new(0, 40, 0, 22)
    Switch.Position = UDim2.new(1, -54, 0.5, -11)
    Switch.BackgroundColor3 = D.Surface2
    Switch.Parent = Card
    corner(Switch, D.RoundFull)
    local sStroke = stroke(Switch, D.Border, 1)

    local Knob = Instance.new("Frame")
    Knob.Size = UDim2.new(0, 16, 0, 16)
    Knob.Position = UDim2.new(0, 3, 0.5, -8)
    Knob.BackgroundColor3 = D.TextDim
    Knob.Parent = Switch
    corner(Knob, D.RoundFull)

    local state = default or false
    local function update()
        if state then
            tween(Switch, 0.2, {BackgroundColor3 = D.Accent})
            tween(Knob, 0.2, {Position = UDim2.new(1, -19, 0.5, -8), BackgroundColor3 = Color3.new(1,1,1)})
            tween(sStroke, 0.2, {Color = D.AccentLt, Transparency = 0})
        else
            tween(Switch, 0.2, {BackgroundColor3 = D.Surface2})
            tween(Knob, 0.2, {Position = UDim2.new(0, 3, 0.5, -8), BackgroundColor3 = D.TextDim})
            tween(sStroke, 0.2, {Color = D.Border, Transparency = 0.5})
        end
    end
    update()

    local hit = Instance.new("TextButton")
    hit.Size = UDim2.new(1, 0, 1, 0)
    hit.BackgroundTransparency = 1
    hit.Text = ""
    hit.Parent = Card
    hit.MouseEnter:Connect(function() tween(cs, 0.15, {Color = D.BorderLit, Transparency = 0.2}) end)
    hit.MouseLeave:Connect(function() tween(cs, 0.15, {Color = D.Border, Transparency = 0.5}) end)
    hit.MouseButton1Click:Connect(function()
        state = not state
        update()
        if callback then callback(state) end
    end)

    return {
        Set = function(v) state = v; update(); if callback then callback(v) end end,
        Get = function() return state end,
        Frame = Card,
    }
end

function Components.Slider(tab, name, min, max, default, callback)
    local Card = Instance.new("Frame")
    Card.Size = UDim2.new(1, 0, 0, 56)
    Card.BackgroundColor3 = D.Surface
    Card.BorderSizePixel = 0
    Card.Parent = tab.Page
    corner(Card, D.RoundSm)
    local cs = stroke(Card, D.Border, 1)
    cs.Transparency = 0.5

    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(0.6, 0, 0, 22)
    lbl.Position = UDim2.new(0, 14, 0, 6)
    lbl.BackgroundTransparency = 1
    lbl.Text = name
    lbl.TextColor3 = D.Text
    lbl.Font = D.FontMed
    lbl.TextSize = 13
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = Card

    local pill = Instance.new("Frame")
    pill.Size = UDim2.new(0, 44, 0, 20)
    pill.Position = UDim2.new(1, -58, 0, 7)
    pill.BackgroundColor3 = D.Surface2
    pill.Parent = Card
    corner(pill, D.RoundFull)
    stroke(pill, D.Border, 1).Transparency = 0.5

    local vLbl = Instance.new("TextLabel")
    vLbl.Size = UDim2.new(1, 0, 1, 0)
    vLbl.BackgroundTransparency = 1
    vLbl.Text = tostring(default)
    vLbl.TextColor3 = D.AccentLt
    vLbl.Font = D.FontBold
    vLbl.TextSize = 11
    vLbl.Parent = pill

    local Bar = Instance.new("Frame")
    Bar.Size = UDim2.new(1, -28, 0, 6)
    Bar.Position = UDim2.new(0, 14, 0, 36)
    Bar.BackgroundColor3 = D.Surface2
    Bar.Parent = Card
    corner(Bar, D.RoundFull)

    local Fill = Instance.new("Frame")
    Fill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    Fill.BackgroundColor3 = D.Accent
    Fill.Parent = Bar
    corner(Fill, D.RoundFull)
    gradient(Fill, D.Accent, D.Accent2, 0)

    local Knob = Instance.new("Frame")
    Knob.Size = UDim2.new(0, 14, 0, 14)
    Knob.Position = UDim2.new(Fill.Size.X.Scale, -7, 0.5, -7)
    Knob.BackgroundColor3 = Color3.new(1, 1, 1)
    Knob.Parent = Bar
    corner(Knob, D.RoundFull)
    stroke(Knob, D.Accent, 2)

    local value = default
    local dragging = false

    local function setValue(v)
        value = math.clamp(v, min, max)
        local pct = (value - min) / (max - min)
        Fill.Size = UDim2.new(pct, 0, 1, 0)
        Knob.Position = UDim2.new(pct, -7, 0.5, -7)
        vLbl.Text = tostring(math.floor(value * 10) / 10)
        if callback then callback(value) end
    end
    setValue(default)

    local hit = Instance.new("TextButton")
    hit.Size = UDim2.new(1, 0, 1, 0)
    hit.BackgroundTransparency = 1
    hit.Text = ""
    hit.Parent = Card
    hit.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            tween(cs, 0.15, {Color = D.AccentLt, Transparency = 0})
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local mx = UIS:GetMouseLocation().X
            local rel = math.clamp((mx - Bar.AbsolutePosition.X) / Bar.AbsoluteSize.X, 0, 1)
            setValue(min + rel * (max - min))
        end
    end)
    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
            tween(cs, 0.15, {Color = D.Border, Transparency = 0.5})
        end
    end)

    return {Set = setValue, Get = function() return value end, Frame = Card}
end

function Components.Button(tab, name, icon, callback)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, 0, 0, 38)
    Btn.BackgroundColor3 = D.Accent
    Btn.Text = ""
    Btn.AutoButtonColor = false
    Btn.Parent = tab.Page
    corner(Btn, D.RoundSm)
    gradient(Btn, D.Accent, D.Accent2, 45)
    local s = stroke(Btn, D.AccentLt, 1)
    s.Transparency = 0.5

    local iconLbl = Instance.new("TextLabel")
    iconLbl.Size = UDim2.new(0, 22, 1, 0)
    iconLbl.Position = UDim2.new(0, 12, 0, 0)
    iconLbl.BackgroundTransparency = 1
    iconLbl.Text = icon or ""
    iconLbl.TextColor3 = Color3.new(1,1,1)
    iconLbl.Font = D.Font
    iconLbl.TextSize = 14
    iconLbl.Parent = Btn

    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, -44, 1, 0)
    lbl.Position = UDim2.new(0, 36, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = name
    lbl.TextColor3 = Color3.new(1,1,1)
    lbl.Font = D.FontBold
    lbl.TextSize = 12
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = Btn

    Btn.MouseEnter:Connect(function()
        tween(Btn, 0.2, {Size = UDim2.new(1, 0, 0, 40)})
        tween(s, 0.2, {Transparency = 0, Color = Color3.new(1,1,1)})
    end)
    Btn.MouseLeave:Connect(function()
        tween(Btn, 0.2, {Size = UDim2.new(1, 0, 0, 38)})
        tween(s, 0.2, {Transparency = 0.5, Color = D.AccentLt})
    end)
    Btn.MouseButton1Click:Connect(function()
        if callback then callback() end
    end)
    return Btn
end

function Components.Input(tab, name, placeholder, callback)
    local Card = Instance.new("Frame")
    Card.Size = UDim2.new(1, 0, 0, 64)
    Card.BackgroundColor3 = D.Surface
    Card.BorderSizePixel = 0
    Card.Parent = tab.Page
    corner(Card, D.RoundSm)
    local cs = stroke(Card, D.Border, 1)
    cs.Transparency = 0.5

    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, -28, 0, 18)
    lbl.Position = UDim2.new(0, 14, 0, 6)
    lbl.BackgroundTransparency = 1
    lbl.Text = name
    lbl.TextColor3 = D.Text
    lbl.Font = D.FontMed
    lbl.TextSize = 12
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = Card

    local Box = Instance.new("TextBox")
    Box.Size = UDim2.new(1, -28, 0, 28)
    Box.Position = UDim2.new(0, 14, 0, 28)
    Box.BackgroundColor3 = D.Bg
    Box.Text = ""
    Box.PlaceholderText = placeholder or ""
    Box.PlaceholderColor3 = D.TextMuted
    Box.TextColor3 = D.Text
    Box.Font = D.Font
    Box.TextSize = 12
    Box.TextXAlignment = Enum.TextXAlignment.Left
    Box.ClearTextOnFocus = false
    Box.Parent = Card
    corner(Box, UDim.new(0, 6))
    local bs = stroke(Box, D.Border, 1)
    bs.Transparency = 0.5
    local ip = Instance.new("UIPadding", Box)
    ip.PaddingLeft = UDim.new(0, 10)

    Box.Focused:Connect(function() tween(bs, 0.15, {Color = D.Accent, Transparency = 0}) end)
    Box.FocusLost:Connect(function()
        tween(bs, 0.15, {Color = D.Border, Transparency = 0.5})
        if callback then callback(Box.Text) end
    end)
    return Box
end

function Components.Label(tab, text, color)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, 0, 0, 24)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = color or D.TextDim
    lbl.Font = D.Font
    lbl.TextSize = 11
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.TextWrapped = true
    lbl.Parent = tab.Page
    return lbl
end

function Components.Card(tab, height)
    local card = Instance.new("Frame")
    card.Size = UDim2.new(1, 0, 0, height or 80)
    card.BackgroundColor3 = D.Surface
    card.BorderSizePixel = 0
    card.Parent = tab.Page
    corner(card, D.RoundSm)
    stroke(card, D.Border, 1).Transparency = 0.5
    return card
end

function Components.Dropdown(tab, name, options, callback)
    local Card = Instance.new("Frame")
    Card.Size = UDim2.new(1, 0, 0, 44)
    Card.BackgroundColor3 = D.Surface
    Card.BorderSizePixel = 0
    Card.ClipsDescendants = true
    Card.Parent = tab.Page
    corner(Card, D.RoundSm)
    local cs = stroke(Card, D.Border, 1)
    cs.Transparency = 0.5

    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(0.5, 0, 0, 44)
    lbl.Position = UDim2.new(0, 14, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = name
    lbl.TextColor3 = D.Text
    lbl.Font = D.FontMed
    lbl.TextSize = 13
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = Card

    local cur = Instance.new("TextLabel")
    cur.Size = UDim2.new(0.5, -20, 0, 44)
    cur.Position = UDim2.new(0.5, 0, 0, 0)
    cur.BackgroundTransparency = 1
    cur.Text = options[1] or "Select"
    cur.TextColor3 = D.AccentLt
    cur.Font = D.FontMed
    cur.TextSize = 12
    cur.TextXAlignment = Enum.TextXAlignment.Right
    cur.Parent = Card

    local listFrame = Instance.new("Frame")
    listFrame.Size = UDim2.new(1, 0, 0, #options * 30)
    listFrame.Position = UDim2.new(0, 0, 0, 44)
    listFrame.BackgroundTransparency = 1
    listFrame.Parent = Card

    local lL = Instance.new("UIListLayout", listFrame)
    lL.Padding = UDim.new(0, 2)

    local opened = false
    local function makeOpt(opt)
        local b = Instance.new("TextButton")
        b.Size = UDim2.new(1, -16, 0, 28)
        b.BackgroundColor3 = D.Bg
        b.Text = "  " .. opt
        b.TextColor3 = D.Text
        b.Font = D.Font
        b.TextSize = 12
        b.TextXAlignment = Enum.TextXAlignment.Left
        b.AutoButtonColor = false
        b.Parent = listFrame
        corner(b, UDim.new(0, 6))
        b.MouseEnter:Connect(function() tween(b, 0.1, {BackgroundColor3 = D.Surface2}) end)
        b.MouseLeave:Connect(function() tween(b, 0.1, {BackgroundColor3 = D.Bg}) end)
        b.MouseButton1Click:Connect(function()
            cur.Text = opt
            opened = false
            tween(Card, 0.2, {Size = UDim2.new(1, 0, 0, 44)})
            if callback then callback(opt) end
        end)
    end
    for _, o in ipairs(options) do makeOpt(o) end

    local hit = Instance.new("TextButton")
    hit.Size = UDim2.new(1, 0, 0, 44)
    hit.BackgroundTransparency = 1
    hit.Text = ""
    hit.Parent = Card
    hit.MouseButton1Click:Connect(function()
        opened = not opened
        if opened then
            tween(Card, 0.25, {Size = UDim2.new(1, 0, 0, 44 + #options * 30 + 8)})
        else
            tween(Card, 0.2, {Size = UDim2.new(1, 0, 0, 44)})
        end
    end)

    return {Get = function() return cur.Text end, Frame = Card}
end

ZyaMe.Components = Components

--// ============================================================
--// WINDOW
--// ============================================================
function ZyaMe:Window(config)
    config = config or {}
    local title = config.Title or "My Hub"
    local subtitle = config.Subtitle or "v1.0"
    local width = config.Width or 620
    local height = config.Height or 440
    local guiName = config.GuiName or "ZyaMeLibWindow"

    pcall(function()
        if CG:FindFirstChild(guiName) then CG[guiName]:Destroy() end
    end)

    local gui = Instance.new("ScreenGui")
    gui.Name = guiName
    gui.ResetOnSpawn = false
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.IgnoreGuiInset = true
    local ok = pcall(function() gui.Parent = CG end)
    if not ok then gui.Parent = lp:WaitForChild("PlayerGui") end

    local Main = Instance.new("Frame")
    Main.Size = UDim2.new(0, width, 0, height)
    Main.Position = UDim2.new(0.5, -width/2, 0.5, -height/2)
    Main.BackgroundColor3 = D.Bg
    Main.BackgroundTransparency = 0.04
    Main.BorderSizePixel = 0
    Main.ClipsDescendants = true
    Main.Parent = gui
    corner(Main, UDim.new(0, 16))
    local ms = stroke(Main, D.Border, 1)
    ms.Transparency = 0.4

    -- Top Bar
    local TopBar = Instance.new("Frame")
    TopBar.Size = UDim2.new(1, 0, 0, 52)
    TopBar.BackgroundColor3 = D.Surface
    TopBar.BackgroundTransparency = 0.3
    TopBar.BorderSizePixel = 0
    TopBar.Parent = Main
    corner(TopBar, UDim.new(0, 16))

    local dot = Instance.new("Frame")
    dot.Size = UDim2.new(0, 8, 0, 8)
    dot.Position = UDim2.new(0, 8, 0, 22)
    dot.BackgroundColor3 = D.Accent
    dot.Parent = TopBar
    corner(dot, D.RoundFull)
    gradient(dot, D.Accent, D.Accent2)

    local tLbl = Instance.new("TextLabel")
    tLbl.Size = UDim2.new(0, 220, 0, 20)
    tLbl.Position = UDim2.new(0, 20, 0, 8)
    tLbl.BackgroundTransparency = 1
    tLbl.Text = title
    tLbl.TextColor3 = D.Text
    tLbl.Font = D.FontBold
    tLbl.TextSize = 16
    tLbl.TextXAlignment = Enum.TextXAlignment.Left
    tLbl.Parent = TopBar

    local sLbl = Instance.new("TextLabel")
    sLbl.Size = UDim2.new(0, 220, 0, 16)
    sLbl.Position = UDim2.new(0, 20, 0, 28)
    sLbl.BackgroundTransparency = 1
    sLbl.Text = subtitle
    sLbl.TextColor3 = D.TextDim
    sLbl.Font = D.FontMed
    sLbl.TextSize = 10
    sLbl.TextXAlignment = Enum.TextXAlignment.Left
    sLbl.Parent = TopBar

    local function topBtn(symbol, color, xOff, onClick)
        local b = Instance.new("TextButton")
        b.Size = UDim2.new(0, 32, 0, 32)
        b.Position = UDim2.new(1, xOff, 0, 10)
        b.BackgroundColor3 = D.Surface2
        b.Text = symbol
        b.TextColor3 = D.TextDim
        b.Font = D.FontBold
        b.TextSize = 14
        b.AutoButtonColor = false
        b.Parent = TopBar
        corner(b, D.RoundFull)
        local s = stroke(b, D.Border, 1)
        b.MouseEnter:Connect(function()
            tween(b, 0.15, {BackgroundColor3 = color, TextColor3 = Color3.new(1,1,1)})
            tween(s, 0.15, {Color = color})
        end)
        b.MouseLeave:Connect(function()
            tween(b, 0.15, {BackgroundColor3 = D.Surface2, TextColor3 = D.TextDim})
            tween(s, 0.15, {Color = D.Border})
        end)
        b.MouseButton1Click:Connect(onClick)
        return b
    end

    local MinBtn = topBtn("–", D.Warning, -80, function() end)
    local CloseBtn = topBtn("✕", D.Danger, -44, function() gui:Destroy() end)

    -- Sidebar
    local Sidebar = Instance.new("Frame")
    Sidebar.Size = UDim2.new(0, 130, 1, -68)
    Sidebar.Position = UDim2.new(0, 12, 0, 56)
    Sidebar.BackgroundColor3 = D.Surface
    Sidebar.BackgroundTransparency = 0.3
    Sidebar.BorderSizePixel = 0
    Sidebar.Parent = Main
    corner(Sidebar, UDim.new(0, 12))
    stroke(Sidebar, D.Border, 1).Transparency = 0.5

    local SideList = Instance.new("UIListLayout")
    SideList.Padding = UDim.new(0, 4)
    SideList.SortOrder = Enum.SortOrder.LayoutOrder
    SideList.HorizontalAlignment = Enum.HorizontalAlignment.Center
    SideList.Parent = Sidebar

    local SidePad = Instance.new("UIPadding")
    SidePad.PaddingTop = UDim.new(0, 10)
    SidePad.PaddingLeft = UDim.new(0, 8)
    SidePad.PaddingRight = UDim.new(0, 8)
    SidePad.Parent = Sidebar

    -- Content
    local Content = Instance.new("Frame")
    Content.Size = UDim2.new(1, -162, 1, -68)
    Content.Position = UDim2.new(0, 152, 0, 56)
    Content.BackgroundTransparency = 1
    Content.Parent = Main

    -- Drag
    local drag, ds, sp
    TopBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            drag = true; ds = input.Position; sp = Main.Position
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if drag and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local d = input.Position - ds
            Main.Position = UDim2.new(sp.X.Scale, sp.X.Offset + d.X, sp.Y.Scale, sp.Y.Offset + d.Y)
        end
    end)
    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then drag = false end
    end)

    local min = false
    local savedSize = Main.Size
    MinBtn.MouseButton1Click:Connect(function()
        min = not min
        if min then
            savedSize = Main.Size
            Main.Size = UDim2.new(0, width, 0, 52)
            Sidebar.Visible = false; Content.Visible = false
        else
            Main.Size = savedSize
            Sidebar.Visible = true; Content.Visible = true
        end
    end)

    -- API
    local Window = {
        Gui = gui, Main = Main, TopBar = TopBar,
        Sidebar = Sidebar, Content = Content,
        Tabs = {},
        Destroy = function() gui:Destroy() end,
        Toggle = function() Main.Visible = not Main.Visible end,
    }

    local CurrentTab = nil

    function Window:Tab(name, icon, order)
        local Btn = Instance.new("TextButton")
        Btn.Size = UDim2.new(1, 0, 0, 38)
        Btn.BackgroundColor3 = D.Surface2
        Btn.BackgroundTransparency = 1
        Btn.Text = ""
        Btn.AutoButtonColor = false
        Btn.LayoutOrder = order or 0
        Btn.Parent = Sidebar
        corner(Btn, D.RoundSm)

        local iconLbl = Instance.new("TextLabel")
        iconLbl.Size = UDim2.new(0, 22, 1, 0)
        iconLbl.Position = UDim2.new(0, 10, 0, 0)
        iconLbl.BackgroundTransparency = 1
        iconLbl.Text = icon or "•"
        iconLbl.TextColor3 = D.TextDim
        iconLbl.Font = D.Font
        iconLbl.TextSize = 14
        iconLbl.Parent = Btn

        local nameLbl = Instance.new("TextLabel")
        nameLbl.Size = UDim2.new(1, -38, 1, 0)
        nameLbl.Position = UDim2.new(0, 34, 0, 0)
        nameLbl.BackgroundTransparency = 1
        nameLbl.Text = name
        nameLbl.TextColor3 = D.TextDim
        nameLbl.Font = D.FontMed
        nameLbl.TextSize = 12
        nameLbl.TextXAlignment = Enum.TextXAlignment.Left
        nameLbl.Parent = Btn

        local ind = Instance.new("Frame")
        ind.Size = UDim2.new(0, 3, 0, 16)
        ind.Position = UDim2.new(0, 0, 0.5, -8)
        ind.BackgroundColor3 = D.Accent
        ind.Visible = false
        ind.Parent = Btn
        corner(ind, D.RoundFull)
        gradient(ind, D.Accent, D.Accent2, 90)

        local Page = Instance.new("ScrollingFrame")
        Page.Size = UDim2.new(1, 0, 1, 0)
        Page.BackgroundTransparency = 1
        Page.BorderSizePixel = 0
        Page.ScrollBarThickness = 3
        Page.ScrollBarImageColor3 = D.Accent
        Page.CanvasSize = UDim2.new(0, 0, 0, 0)
        Page.AutomaticCanvasSize = Enum.AutomaticSize.Y
        Page.Visible = false
        Page.Parent = Content

        local PL = Instance.new("UIListLayout", Page)
        PL.Padding = UDim.new(0, 8)
        PL.SortOrder = Enum.SortOrder.LayoutOrder
        local PP = Instance.new("UIPadding", Page)
        PP.PaddingRight = UDim.new(0, 6)
        PP.PaddingBottom = UDim.new(0, 10)
        PP.Parent = Page

        local tab = {Button = Btn, Page = Page, Indicator = ind, NameLabel = nameLbl, Name = name}

        Btn.MouseEnter:Connect(function()
            if CurrentTab ~= tab then tween(Btn, 0.15, {BackgroundTransparency = 0.5}) end
        end)
        Btn.MouseLeave:Connect(function()
            if CurrentTab ~= tab then tween(Btn, 0.15, {BackgroundTransparency = 1}) end
        end)
        Btn.MouseButton1Click:Connect(function()
            for _, t in pairs(Window.Tabs) do
                t.Page.Visible = false
                t.Indicator.Visible = false
                tween(t.Button, 0.2, {BackgroundTransparency = 1})
                t.NameLabel.TextColor3 = D.TextDim
            end
            Btn.BackgroundTransparency = 0
            Btn.BackgroundColor3 = D.Surface2
            nameLbl.TextColor3 = D.Text
            ind.Visible = true
            Page.Visible = true
            CurrentTab = tab
        end)

        table.insert(Window.Tabs, tab)
        if #Window.Tabs == 1 then
            task.defer(function()
                Btn.BackgroundTransparency = 0
                Btn.BackgroundColor3 = D.Surface2
                nameLbl.TextColor3 = D.Text
                ind.Visible = true
                Page.Visible = true
                CurrentTab = tab
            end)
        end

        -- Attach component methods
        local tabApi = {
            Page = Page, Button = Btn,
        }
        function tabApi:Section(text) return Components.Section(self, text) end
        function tabApi:Toggle(name, desc, default, cb) return Components.Toggle(self, name, desc, default, cb) end
        function tabApi:Slider(name, min, max, default, cb) return Components.Slider(self, name, min, max, default, cb) end
        function tabApi:Button(name, icon, cb) return Components.Button(self, name, icon, cb) end
        function tabApi:Input(name, placeholder, cb) return Components.Input(self, name, placeholder, cb) end
        function tabApi:Label(text, color) return Components.Label(self, text, color) end
        function tabApi:Card(height) return Components.Card(self, height) end
        function tabApi:Dropdown(name, options, cb) return Components.Dropdown(self, name, options, cb) end

        return tabApi
    end

    return Window
end

--// ============================================================
--// FLOATS
--// ============================================================
function ZyaMe:Floats(config)
    config = config or {}
    local guiName = config.GuiName or "ZyaMeLibFloats"

    pcall(function()
        if CG:FindFirstChild(guiName) then CG[guiName]:Destroy() end
    end)

    local gui = Instance.new("ScreenGui")
    gui.Name = guiName
    gui.ResetOnSpawn = false
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.IgnoreGuiInset = true
    local ok = pcall(function() gui.Parent = CG end)
    if not ok then gui.Parent = lp:WaitForChild("PlayerGui") end

    local Registry = {}
    local State = {
        Lock = false,
        Rainbow = false,
        RainbowSpeed = 1,
        Mode = false,
    }

    local api = {
        Gui = gui,
        Registry = Registry,
        State = State,
    }

    function api:Create(cfg)
        cfg = cfg or {}
        local Btn = Instance.new("TextButton")
        Btn.Name = cfg.Name or "FloatBtn"
        Btn.Size = cfg.Size or UDim2.new(0, 70, 0, 70)
        Btn.Position = cfg.Position or UDim2.new(0, 20, 0, 20)
        Btn.BackgroundColor3 = D.Bg
        Btn.BackgroundTransparency = 0.1
        Btn.Text = cfg.Text or "BTN"
        Btn.TextColor3 = D.Text
        Btn.Font = D.FontBold
        Btn.TextSize = cfg.TextSize or 12
        Btn.AutoButtonColor = false
        Btn.Visible = cfg.Visible == true
        Btn.Parent = gui
        corner(Btn, (cfg.Round == false) and D.RoundSm or D.RoundFull)
        local s = stroke(Btn, cfg.Color or D.Accent, 2)
        s.Transparency = 0.1

        local inner = Instance.new("ImageLabel")
        inner.Size = UDim2.new(1, 0, 1, 0)
        inner.BackgroundTransparency = 1
        inner.Image = "rbxassetid://5028857472"
        inner.ImageColor3 = cfg.Color or D.Accent
        inner.ImageTransparency = 0.85
        inner.Parent = Btn

        Btn.MouseEnter:Connect(function()
            tween(Btn, 0.2, {BackgroundTransparency = 0})
            tween(s, 0.2, {Thickness = 3})
            tween(inner, 0.2, {ImageTransparency = 0.6})
        end)
        Btn.MouseLeave:Connect(function()
            tween(Btn, 0.2, {BackgroundTransparency = 0.1})
            tween(s, 0.2, {Thickness = 2})
            tween(inner, 0.2, {ImageTransparency = 0.85})
        end)

        local drag, ds, sp, moved = false, nil, nil, false
        Btn.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                if State.Lock then return end
                drag = true; moved = false
                ds = input.Position; sp = Btn.Position
            end
        end)
        UIS.InputChanged:Connect(function(input)
            if drag and not State.Lock and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                local d = input.Position - ds
                if math.abs(d.X) > 3 or math.abs(d.Y) > 3 then moved = true end
                Btn.Position = UDim2.new(sp.X.Scale, sp.X.Offset + d.X, sp.Y.Scale, sp.Y.Offset + d.Y)
            end
        end)
        UIS.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                if drag and not moved and cfg.Callback then cfg.Callback() end
                drag = false
            end
        end)
        Btn.MouseButton1Click:Connect(function()
            if State.Lock and cfg.Callback then cfg.Callback() end
        end)

        local entry = {
            Name = cfg.Name, Button = Btn, Stroke = s,
            OriginalText = cfg.Text, OriginalColor = cfg.Color or D.Accent,
            OriginalSize = cfg.Size or UDim2.new(0, 70, 0, 70),
            RainbowEnabled = false, RainbowConn = nil,
            Visible = Btn.Visible,
        }
        table.insert(Registry, entry)
        return Btn, entry
    end

    function api:SetAllVisible(state)
        State.Mode = state
        for _, e in ipairs(Registry) do
            e.Button.Visible = state
            e.Visible = state
        end
    end

    function api:SetLock(state)
        State.Lock = state
    end

    function api:SetRainbow(state, entry)
        State.Rainbow = state
        if entry then
            entry.RainbowEnabled = state
            if state then
                if entry.RainbowConn then entry.RainbowConn:Disconnect() end
                local hue = 0
                entry.RainbowConn = RS.Heartbeat:Connect(function(dt)
                    if not entry.RainbowEnabled then return end
                    hue = (hue + dt * 0.3 * State.RainbowSpeed) % 1
                    local c = HSLtoRGB(hue, 1, 0.6)
                    if entry.Stroke and entry.Stroke.Parent then entry.Stroke.Color = c end
                end)
            else
                if entry.RainbowConn then entry.RainbowConn:Disconnect(); entry.RainbowConn = nil end
                entry.Stroke.Color = entry.OriginalColor
            end
        else
            for _, e in ipairs(Registry) do
                e.RainbowEnabled = state
                if state then
                    if e.RainbowConn then e.RainbowConn:Disconnect() end
                    local hue = 0
                    local ee = e
                    e.RainbowConn = RS.Heartbeat:Connect(function(dt)
                        if not ee.RainbowEnabled then return end
                        hue = (hue + dt * 0.3 * State.RainbowSpeed) % 1
                        local c = HSLtoRGB(hue, 1, 0.6)
                        if ee.Stroke and ee.Stroke.Parent then ee.Stroke.Color = c end
                    end)
                else
                    if e.RainbowConn then e.RainbowConn:Disconnect(); e.RainbowConn = nil end
                    e.Stroke.Color = e.OriginalColor
                end
            end
        end
    end

    function api:SetRainbowSpeed(v)
        State.RainbowSpeed = v
    end

    function api:Destroy()
        gui:Destroy()
        for _, e in ipairs(Registry) do
            if e.RainbowConn then e.RainbowConn:Disconnect() end
        end
    end

    return api
end

--// ============================================================
--// MISC UTILS
--// ============================================================
function ZyaMe:Notify(title, text, duration)
    notify(title, text, duration)
end

return ZyaMe
