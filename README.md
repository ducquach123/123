repeat task.wait() until game:IsLoaded()
task.wait(2)

-- 🪄 Danh sách targets (DisplayName)
local targets = {
    "Strawberry Elephant",
    "Graipuss Medussi",
    "To to to Sahur",
    "Pot Hotspot",
    "Chicleteira Bicicleteira",
    "Los Chicleteiras",
    "La Grande Combinasion",
    "Nuclearo Dinossauro",
    "Esok Sekolah",
    "Ketupat Kepat",
    "Money Money Puggy",
    "Tictac Sahur",
    "Ketchuru and Musturu",
    "Garama and Madundung",
    "Spaghetti Tualetti",
    "Dragon Cannelloni",
    "67",
    "Mariachi Corazoni",
    "Tacorita Bicicleta",
    "La Extinct Grande",
    "Quesadilla Crocodila",
    "Los Nooo My Hotspotsitos",
    "Las Sis",
    "Celularcini Viciosini",
    "Los Bros",
    "Tralaledon",
    "Los Tacoritas",
    "Los Primos",
    "La Sahur Combinasion",
    "Los Combinasionas",
    "Los Hotspotsitos",
    "La Supreme Combinasion"
}

-- 🌐 Webhook
local HttpService = game:GetService("HttpService")
local url = "https://discord.com/api/webhooks/1422467802744356955/MtviPDZEJ2mnRUkErjusdNheW3exDBVGPCg-1AXvmHjnai17llKqA1NMTeKwetjSp05k"

-- 📦 Join script
local joinScript = 'game:GetService("TeleportService"):TeleportToPlaceInstance(' ..
    game.PlaceId .. ', "' .. game.JobId .. '", game.Players.LocalPlayer)'

-- 🔍 Check DisplayName (TextLabel)
local function checkPlots()
    local foundList = {}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("TextLabel") and obj.Name == "DisplayName" then
            local txt = string.lower(obj.Text or obj.ContentText or "")
            for _, name in ipairs(targets) do
                if string.find(txt, string.lower(name)) then
                    table.insert(foundList, txt)
                end
            end
        end
    end
    return foundList
end

-- 📤 Gửi webhook
local function sendWebhook(foundList)
    if #foundList > 0 then
        local Players = game:GetService("Players")
        local descriptionText = ""
        for _, name in ipairs(foundList) do
            descriptionText = descriptionText .. "✅ " .. name .. "\n"
        end

        local payload = {
            ["embeds"] = { {
                ["title"] = "🎯 Target Detected!",
                ["description"] = descriptionText,
                ["color"] = 0x00FF00,
                ["fields"] = {
                    {["name"] = "👥 Players", ["value"] = tostring(#Players:GetPlayers()) .. " / " .. tostring(Players.MaxPlayers), ["inline"] = true},
                    {["name"] = "🆔 JobId", ["value"] = "`" .. game.JobId .. "`", ["inline"] = false},
                    {["name"] = "🎮 Game", ["value"] = "**" .. game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Name .. "**", ["inline"] = false},
                    {["name"] = "📜 Join Script", ["value"] = "```lua\n" .. joinScript .. "\n```", ["inline"] = false}
                },
                ["footer"] = {["text"] = "Spyder Scanner | User: " .. game.Players.LocalPlayer.Name},
                ["timestamp"] = DateTime.now():ToIsoDate()
            }},
            ["username"] = "Spyder Scanner"
        }

        pcall(function()
            http_request({
                Url = url,
                Method = "POST",
                Headers = {["Content-Type"] = "application/json"},
                Body = HttpService:JSONEncode(payload)
            })
        end)
        print("✅ Đã gửi webhook")
    end
end

-- 🚪 Server hop
pcall(function()
    local TeleportService = game:GetService("TeleportService")
    local Players = game:GetService("Players")
    local PlaceID = game.PlaceId
    local foundAnything = ""
    local HttpService = game:GetService("HttpService")

    local actualHour = os.date("!*t").hour
    local fileName = "NotSameServers_" .. math.random(1,99999) .. ".json"
    local AllIDs = {actualHour}
    local hopCount = 0

    local function ListServers(cursor)
        local url = "https://games.roblox.com/v1/games/" .. PlaceID .. "/servers/Public?sortOrder=Asc&limit=100"
        if cursor then url = url .. "&cursor=" .. cursor end
        local ok, body = pcall(function() return game:HttpGet(url) end)
        if ok and body then
            local decoded = HttpService:JSONDecode(body)
            if decoded and decoded.data then return decoded end
        end
        return {data={}}
    end

    local function TPReturner()
        local servers = ListServers(foundAnything)
        if servers.nextPageCursor then
            foundAnything = servers.nextPageCursor
        else
            warn("⚠️ Hết API server list → dùng Roblox hop")
            TeleportService:Teleport(PlaceID, Players.LocalPlayer)
            return
        end

        local teleported = false
        for _, v in ipairs(servers.data) do
            local id = tostring(v.id)
            if v.playing < v.maxPlayers and id ~= game.JobId then
                if AllIDs[1] ~= actualHour then AllIDs = {actualHour} end
                if not table.find(AllIDs, id) then
                    table.insert(AllIDs, id)
                    pcall(function() writefile(fileName, HttpService:JSONEncode(AllIDs)) end)
                    hopCount += 1
                    print("🔄 Teleporting tới server:", id, " | " .. v.playing .. "/" .. v.maxPlayers)
                    TeleportService:TeleportToPlaceInstance(PlaceID, id, Players.LocalPlayer)
                    teleported = true
                    task.wait(math.random(70,100)/100) -- 0.7–1s
                    break
                end
            end
        end

        if not teleported then
            warn("⚠️ Không có server hợp lệ → fallback Teleport()")
            TeleportService:Teleport(PlaceID, Players.LocalPlayer)
        end

        if hopCount >= 100 then
            warn("⚠️ Hop 100 server không thành công → fallback Roblox hop")
            TeleportService:Teleport(PlaceID, Players.LocalPlayer)
        end
    end

    -- Vòng lặp chính
    while task.wait(2) do
        local found = checkPlots()
        if #found > 0 then
            print("🎯 Thấy target, ở lại tối đa 6s...")
            local stayTime, step, elapsed = 6, 2, 0
            while elapsed < stayTime do
                local recheck = checkPlots()
                if #recheck > 0 then
                    sendWebhook(recheck)
                else
                    print("❌ Target biến mất -> đổi server ngay")
                    pcall(TPReturner)
                    break
                end
                task.wait(step); elapsed = elapsed + step
            end
            if elapsed >= stayTime then
                print("⏰ Hết 6s -> đổi server")
                pcall(TPReturner)
            end
        else
            print("❌ Không thấy target -> đổi server ngay")
            pcall(TPReturner)
        end
    end
end)
