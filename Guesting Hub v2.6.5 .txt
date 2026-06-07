local Translations = {
    ["Info/Updates"] = "信息/更新",
    ["Thanks FOR using: ) Update 2.6.5"] = "感谢使用 :) | 更新 2.6.5",
    ["YOOO"] = "哟",
    ["Alr"] = "好的",
    ["Search"] = "搜索",
    ["DiscordYaps"] = "Discord | 闲聊",
    ["Member Count:6596"] = "成员数：6596",
    ["Online Count: 572"] = "在线数：572",
    ["Copy Discord Invite"] = "复制Discord链接",
    ["JOIN DISCORD SERVERRR"] = "加入 Discord 服务器",
    ["Updates"] = "更新",
    ["Info"] = "信息",
    ["By-Veux"] = "作者：Veux",
    ["Make sure your Ping Is below 160 This is justV1 so u may suck Bugs!  ?  !  ?  !  ALSO FORCOOLKID u need-100 ping btw"] = "请确保你的 Ping 值低于 160。这仅仅是 V1 版本，因此你可能会遇到 Bug！？！ 另外，对于 COOLKID 你需要 -100 的 Ping 值，顺便说一句。",
    ["Latest UpdatesPATCH UPDATEAdded some things and fixedFixed the aimbots black screenFixed the M1 black screenFixed crystal surfacesAdded silent aimUI Changes -Now auto punch and delay are in the autoblockScroll down and you will seeMade all aimbots in a drop-down style thingy.Added config code generator and import in anew tab called Config"] = "最新更新/n· 补丁更新 -/n 添加了一些内容并进行了修复 -/n· 修复了自瞄的黑屏问题/n· 修复了 M1 的黑屏问题/n· 修复了水晶挤压问题/n· 添加了静默自瞄/n· 界面更改 -/n现在自动拳击和延迟位于自动拦截中向下滚动即可看到。/n· 将所有自瞄选项改为下拉菜单样式。/n· 添加了配置代码生成器和导入功能，位于名为“配置”的新标签页中",
    ["Fixed stamina visualizer not being exposed.Bypass anti cheat of Inf Boxy ColaYou may see some obvious effect during thespeed but ignore it!  !  !Fixed custom LMS.Made all chance skin and other Shedletsky andJane Doe skin and guest aimbots work inaimbotTHE SKINS ID UPDATED IN Forsaken 4.0.1UpdateIf they added more skins above that, thosenew skins may not work!Added new skins to work in auto block.And other etc things"] = "修复了体力值显示不准确的问题。/n绕过无限可乐的反作弊机制在加速过程中你可能会看到一些奇怪的特效，但请忽略它！！！/n修复了自定义 LMS。/n使所有随机皮肤、其他谢德列斯基和约翰多皮肤以及访客的自瞄都能在自瞄功能中生效/n这些皮肤 ID 已在 Forsaken 4.0.1 更新中进行了更新/n如果他们在该版本之上添加了更多新皮肤，这些新皮肤可能无法生效！/n添加了新皮肤以使其在自动拦截中生效。/n以及其他等等内容。"，
    [""] = "",
}

local function translateText(text)
    if not text or type(text) ~= "string" then 
        return text 
    end
    
    -- 直接匹配整个文本
    if Translations[text] then
        return Translations[text]
    end
    
    -- 部分匹配替换
    for en, cn in pairs(Translations) do
        if en ~= "" and text:find(en, 1, true) then
            return text:gsub(en, cn)
        end
    end
    
    return text
end

local function setupTranslationEngine()
    local success, err = pcall(function()
        -- 尝试使用元表劫持
        if getrawmetatable and setreadonly and newcclosure then
            local oldIndex = getrawmetatable(game).__newindex
            setreadonly(getrawmetatable(game), false)
            
            getrawmetatable(game).__newindex = newcclosure(function(t, k, v)
                if (t:IsA("TextLabel") or t:IsA("TextButton") or t:IsA("TextBox")) and k == "Text" then
                    v = translateText(tostring(v))
                end
                return oldIndex(t, k, v)
            end)
            
            setreadonly(getrawmetatable(game), true)
            return true
        else
            error("Roblox环境不支持元表操作")
        end
    end)
    
    if not success then
        warn("元表劫持失败:", err)
       
        -- 回退方案：扫描和翻译所有现有的GUI元素
        local translated = {}
        local function scanAndTranslate()
            -- 扫描CoreGui
            pcall(function()
                for _, gui in ipairs(game:GetService("CoreGui"):GetDescendants()) do
                    if (gui:IsA("TextLabel") or gui:IsA("TextButton") or gui:IsA("TextBox")) and not translated[gui] then
                        local text = gui.Text
                        if text and text ~= "" then
                            local translatedText = translateText(text)
                            if translatedText ~= text then
                                gui.Text = translatedText
                                translated[gui] = true
                            end
                        end
                    end
                end
            end)
            
            -- 扫描PlayerGui
            pcall(function()
                local player = game:GetService("Players").LocalPlayer
                if player and player:FindFirstChild("PlayerGui") then
                    for _, gui in ipairs(player.PlayerGui:GetDescendants()) do
                        if (gui:IsA("TextLabel") or gui:IsA("TextButton") or gui:IsA("TextBox")) and not translated[gui] then
                            local text = gui.Text
                            if text and text ~= "" then
                                local translatedText = translateText(text)
                                if translatedText ~= text then
                                    gui.Text = translatedText
                                    translated[gui] = true
                                end
                            end
                        end
                    end
                end
            end)
        end
        
        -- 设置新元素监听器
        local function setupDescendantListener(parent)
            parent.DescendantAdded:Connect(function(descendant)
                if descendant:IsA("TextLabel") or descendant:IsA("TextButton") or descendant:IsA("TextBox") then
                    task.wait(0.1)
                    pcall(function()
                        local text = descendant.Text
                        if text and text ~= "" then
                            local translatedText = translateText(text)
                            if translatedText ~= text then
                                descendant.Text = translatedText
                            end
                        end
                    end)
                end
            end)
        end
        
        pcall(function()
            setupDescendantListener(game:GetService("CoreGui"))
        end)
        
        pcall(function()
            local player = game:GetService("Players").LocalPlayer
            if player and player:FindFirstChild("PlayerGui") then
                setupDescendantListener(player.PlayerGui)
            end
        end)
        
        -- 初始扫描
        scanAndTranslate()
        
        -- 定期扫描
        spawn(function()
            while true do
                scanAndTranslate()
                task.wait(3)
            end
        end)
    end
end

task.wait(2)

setupTranslationEngine()

local success, err = pcall(function()
    -- 这下面填加载外部脚本
    loadstring(game:HttpGet("https://raw.githubusercontent.com/LolnotaKid/project/refs/heads/main/AutoBLOCKKKWAHV1"))()
end)

if not success then
    warn("加载失败:", err)
end