repeat task.wait() until game:IsLoaded()
task.wait(2)

-- 🪄 Danh sách targets (DisplayName.LocalizedText)
local targets = {
    "te te te sahur", -- 250K
}

-- 🌐 Webhook
local HttpService = game:GetService("HttpService")
local url = "https://discord.com/api/webhooks/1422467802744356955/MtviPDZEJ2mnRUkErjusdNheW3exDBVGPCg-1AXvmHjnai17llKqA1NMTeKwetjSp05k"

-- 📦 Join script để gửi lên Discord
local joinScript = 'game:GetService("TeleportService"):TeleportToPlaceInstance(' ..
    game.PlaceId .. ', "' .. game.JobId .. '", game.Players.LocalPlayer)'

-- 🔍 Check targets trong DisplayName (GUI)
local function checkPlots()
    local foundList = {}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("TextLabel") and obj.Name == "DisplayName" then
            local txt = string.lower(obj.LocalizedText or "")
            for _, name in ipairs(targets) do
                if string.find(txt, string.lower(name)) then
                    table.insert(foundList, txt)
                end
            end
        end
    end
    return foundList
end

-- 📤 Gửi danh sách tìm thấy lên Discord (embed đẹp)
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
                    {
                        ["name"] = "👥 Players",
                        ["value"] = tostring(#Players:GetPlayers()) .. " / " .. tostring(Players.MaxPlayers),
                        ["inline"] = true
                    },
                    {
                        ["name"] = "🆔 JobId",
                        ["value"] = "`" .. game.JobId .. "`",
                        ["inline"] = false
                    },
                    {
                        ["name"] = "🎮 Game",
                        ["value"] = "**" .. game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Name .. "**",
                        ["inline"] = false
                    },
                    {
                        ["name"] = "📜 Join Script",
                        ["value"] = "```lua\n" .. joinScript .. "\n```",
                        ["inline"] = false
                    }
                },
                ["footer"] = {
                    ["text"] = "Spyder Scanner | User: " .. game.Players.LocalPlayer.Name
                },
                ["timestamp"] = DateTime.now():ToIsoDate()
            }},
            ["username"] = "Spyder Scanner",
            ["avatar_url"] = "https://i.imgur.com/xxxx.png"
        }

        pcall(function()
            http_request({
                Url = url,
                Method = "POST",
                Headers = {["Content-Type"] = "application/json"},
                Body = HttpService:JSONEncode(payload)
            })
        end)

        print("✅ Đã gửi webhook (targets + player + info)")
    end
end

-- 🚪 Server hop NÂNG CẤP với RETRY
local TeleportService = game:GetService("TeleportService")
local Players = game:GetService("Players")
local PlaceID = game.PlaceId

local function smartServerHop()
    local maxRetries = 5  -- Thử tối đa 5 lần
    local retryCount = 0
    local originalJobId = game.JobId  -- Lưu JobId ban đầu
    
    while retryCount < maxRetries do
        retryCount = retryCount + 1
        print("🔄 Lần thử", retryCount, "/", maxRetries)
        
        -- Delay ngẫu nhiên để rải đều 14 acc
        task.wait(math.random(1, 8))
        
        local success, errorMsg = pcall(function()
            local servers = TeleportService:GetServersAsync(PlaceID)
            
            local validServers = {}
            for _, server in ipairs(servers) do
                -- Loại server hiện tại và server gần đầy
                if server.Id ~= originalJobId and server.Playing < server.MaxPlayers - 2 then
                    table.insert(validServers, server)
                end
            end
            
            if #validServers > 0 then
                -- Chọn random để phân tán
                local randomServer = validServers[math.random(1, #validServers)]
                print("🔄 Đang hop tới server:", randomServer.Id)
                TeleportService:TeleportToPlaceInstance(PlaceID, randomServer.Id, Players.LocalPlayer)
                
                -- Đợi 3 giây để check xem có thực sự đổi server không
                task.wait(3)
                
                -- Kiểm tra xem đã đổi server chưa
                if game.JobId ~= originalJobId then
                    print("✅ Đã hop thành công!")
                    return true  -- Hop thành công, thoát function
                else
                    warn("⚠️ Vẫn ở server cũ, thử lại...")
                    -- Tiếp tục loop để retry
                end
            else
                warn("⚠️ Không tìm thấy server phù hợp, thử lại...")
                task.wait(2)
                -- Tiếp tục loop để retry
            end
        end)
        
        if not success then
            warn("❌ Lỗi lần", retryCount, ":", errorMsg)
            task.wait(2)
        end
    end
    
    -- Nếu hết retry mà vẫn không hop được, dùng fallback
    warn("🆘 Đã thử", maxRetries, "lần không thành công, dùng Teleport thường")
    pcall(function()
        TeleportService:Teleport(PlaceID, Players.LocalPlayer)
    end)
end

-- 🔁 Main loop
while task.wait(3) do
    local found = checkPlots()

    if #found > 0 then
        print("🎯 Thấy target, ở lại tối đa 6s...")
        local stayTime = 6
        local step = 2
        local elapsed = 0

        while elapsed < stayTime do
            local recheck = checkPlots()
            if #recheck > 0 then
                sendWebhook(recheck)
            else
                print("❌ Target biến mất -> đổi server ngay")
                smartServerHop()
                break
            end
            task.wait(step)
            elapsed = elapsed + step
        end

        if elapsed >= stayTime then
            print("⏰ Hết 6s -> đổi server")
            smartServerHop()
        end
    else
        print("❌ Không thấy target -> đổi server ngay")
        smartServerHop()
    end
end
