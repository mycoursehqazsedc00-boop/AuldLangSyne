
## AuldLangSyne.lua
```lua
local L = AceLibrary("AceLocale-2.2"):new("AuldLangSyne")
AuldLangSyne = AceLibrary("AceAddon-2.0"):new("AceEvent-2.0", "AceHook-2.1", "AceDebug-2.0", "AceConsole-2.0", "AceDB-2.0", "FuBarPlugin-2.0")

local deformat = AceLibrary("Deformat-2.0")
local tablet = AceLibrary("Tablet-2.0")
local BC = AceLibrary("Babble-Class-2.2")

local defaults_realm = {
	chars = {
	},
	ignore = {},
	guild = {},
}
local defaults_char = {
	friends = {
		["*"] = {},
	},
}
local defaults_profile = {
	show = {
		pvp = true,
		pvpnum = false,
		guild = true,
		color = {
			name = true,
			class = true,
		},
		compact = false,
		clevels = true,
		intooltip = true,
		onlogon = true,
		onwho = true,
	},
}

local options = {
	type = "group",
	name = L["AuldLangSyne"],
	desc = L["Options for AuldLangSyne."],
	handler = AuldLangSyne,
	args = {
		data = {
			type = "group", name = L["data"], desc = L["Random data-management functions"], handler = AuldLangSyne,
			args = {
				clear = {
					name = L["clear"], type = "execute",
					desc = L["Clear stored data"],
					func = "ClearData",
				},
				dump = {
					name = L["dump"], type = "execute",
					desc = L["Print stored data"],
					func = "DumpData",
				},
				ctimport = {
					name = L["ctimport"], type = "execute",
					desc = L["Import notes for this realm from CT_PlayerNotes"],
					func = "CTImport",
				},
			},
		},
		backup = {
			name = L["backup"], type = "group",
			desc = L["Options for backing up and restoring your friends list"], handler = AuldLangSyne,
			args = {
				save = {
					name = L["save"], type = "text", get = false,
					desc = L["Take a snapshot of your current friends list"],
					set = "SaveFriends", validate = {"1","2","3","4","5"},
				},
				load = {
					name = L["load"], type = "text", get = false,
					desc = L["Restore a snapshot of your friends list, wiping out your current friends list"],
					set = "LoadFriends", validate = {"1","2","3","4","5","undo"},
				},
			},
		},
		show = {
			type = "group",
			name = L["show"], desc = L["Choose what information to display"], handler = AuldLangSyne,
			args = {
				pvp = {
					name = "PVP", type = "toggle",
					desc = L["Turns display of PVP ranks on and off."],
					get = function() return AuldLangSyne.db.profile.show.pvp end,
					set = function(t)
						AuldLangSyne.db.profile.show.pvp = t
					end,
				},
				pvpnum = {
					name = L["PVP rank numbers"], type = "toggle",
					desc = L["Turns display of PVP rank numbers on and off."],
					get = function() return AuldLangSyne.db.profile.show.pvpnum end,
					set = function(t)
						AuldLangSyne.db.profile.show.pvpnum = t
					end,
				},
				guild = {
					name = L["Guild"], type = "toggle",
					desc = L["Turns display of guild membership on and off."],
					get = function() return AuldLangSyne.db.profile.show.guild end,
					set = function(t)
						AuldLangSyne.db.profile.show.guild = t
					end,
				},
				color = {
					type = "group", name = L["Colors"], desc = L["Choose which elements to color"], handler = AuldLangSyne,
					args = {
						name = {
							name = L["Name"], type = "toggle",
							desc = L["Turns coloring names by gender on and off."],
							get = function() return AuldLangSyne.db.profile.show.color.name end,
							set = function(t)
								AuldLangSyne.db.profile.show.color.name = t
							end,
						},
						class = {
							name = L["Class"], type = "toggle",
							desc = L["Turns coloring class names on and off."],
							get = function() return AuldLangSyne.db.profile.show.color.class end,
							set = function(t)
								AuldLangSyne.db.profile.show.color.class = t
							end,
						}
					},
				},
				compact = {
					name = L["Compact mode"], type = "toggle",
					desc = L["Turns compact friends list on and off."],
					get = function() return AuldLangSyne.db.profile.show.compact end,
					set = function(t)
						AuldLangSyne.db.profile.show.compact = t
					end,
				},
				clevels = {
					name = L["Compact mode levels"], type = "toggle",
					desc = L["Turns display of levels in compact mode on and off."],
					get = function() return AuldLangSyne.db.profile.show.clevels end,
					set = function(t)
						AuldLangSyne.db.profile.show.clevels = t
					end,
				},
				intooltip = {
					name = L["Note in player tooltip"], type = "toggle",
					desc = L["Turns display of player notes in tooltips on and off."],
					get = function() return AuldLangSyne.db.profile.show.intooltip end,
					set = function(t)
						AuldLangSyne.db.profile.show.intooltip = t
					end,
				},
				onlogon = {
					name = L["Note on logon"], type = "toggle",
					desc = L["Turns display of player notes on logon on and off."],
					get = function() return AuldLangSyne.db.profile.show.onlogon end,
					set = function(t)
						AuldLangSyne.db.profile.show.onlogon = t
					end,
				},
				onwho = {
					name = L["Note on /who"], type = "toggle",
					desc = L["Turns display of player notes when you /who them on and off."],
					get = function() return AuldLangSyne.db.profile.show.onwho end,
					set = function(t)
						AuldLangSyne.db.profile.show.onwho = t
					end,
				},
			},
		},
	},
}

function AuldLangSyne:OnInitialize()
	--self:SetDebugging(true)

	self:RegisterDB("AuldLangSyneDB")
	self:RegisterDefaults("realm", defaults_realm)
	self:RegisterDefaults("char", defaults_char)
	self:RegisterDefaults("profile", defaults_profile)

	self:RegisterChatCommand({"/AuldLangSyne", "/auld"}, options)
  -- Fubar
  self.OnMenuRequest = options
  if not FuBar then
    self.OnMenuRequest.args.hide.guiName = L["Hide minimap icon"]
    self.OnMenuRequest.args.hide.desc = L["Hide minimap icon"]
  end
	-- We will need many buttons.
	self.friendbuttons, self.ignorebuttons, self.guildbuttons = {}, {}, {}
	for i=1, FRIENDS_TO_DISPLAY, 1 do
		table.insert(self.friendbuttons, self:CreateButton("AuldLangSyne_FriendNote"..i, getglobal("FriendsFrameFriendButton"..i), "RIGHT", "RIGHT", -5, 0))
		self.friendbuttons[i].type = "friend"
	end
	for i=1, IGNORES_TO_DISPLAY, 1 do
		table.insert(self.ignorebuttons, self:CreateButton("AuldLangSyne_IgnoreNote"..i, getglobal("FriendsFrameIgnoreButton"..i), "RIGHT", "RIGHT", -5, 0))
		self.ignorebuttons[i].type = "ignore"
	end
	for i=1, GUILDMEMBERS_TO_DISPLAY, 1 do
		table.insert(self.guildbuttons, self:CreateButton("AuldLangSyne_GuildNote"..i, getglobal("GuildFrameButton"..i), "RIGHT", "RIGHT", 0, 0))
		self.guildbuttons[i].type = "guild"
	end

	self.colors_sex = {
		[MALE] = "90b0f0",
		[FEMALE] = "f090c4",
	}

	self.editFrame = self:CreateEditFrame()

	if (not self.db.realm.version) or self.db.realm.version ~= self.version then
		--DB upgrade!
		self:Print(L["Detected an older database version.  Upgrading!"])
		local v,_
		if self.db.realm.version then
			-- We rely on the version being of the form x.x.svn_rev
			_,_,v = string.find(self.db.realm.version, "%d+\.%d+\.(%d+)")
			v = tonumber(v)
		end

		if (not v) or v <= 4556 then
			--Strip out the location information from the DB.
			for name,info in pairs(self.db.realm.chars) do
				-- We stopped recording the area.
				info.area = nil
			end
		end

		if (not v) or v <= 4807 then
			for name,info in pairs(self.db.realm.chars) do
				-- We're now storing the output of UnitPVPRank in .rank instead of UnitPVPName in .pvp.  Clear out the old field.
				info.pvp = nil
			end
		end
	end

	self.db.realm.version = self.version

	--Stolen from AceDB.
	local _,race = UnitRace("player")
	if race == "Orc" or race == "Scourge" or race == "Troll" or race == "Tauren" then
		self.faction = "0"
	else
		self.faction = "1"
	end

	self.undo = {}
end

---------
-- FuBar
---------
AuldLangSyne.hasIcon = "Interface\\FriendsFrame\\FriendsFrameScrollIcon"
AuldLangSyne.title = L["AuldLangSyne"]
AuldLangSyne.defaultMinimapPosition = 225
AuldLangSyne.defaultPosition = "LEFT"
AuldLangSyne.cannotDetachTooltip = true
AuldLangSyne.tooltipHiddenWhenEmpty = false
AuldLangSyne.hideWithoutStandby = true
AuldLangSyne.independentProfile = true

function AuldLangSyne:OnEnable()
	self:RegisterEvent("FRIENDLIST_UPDATE", "UpdateFriends")
	self:SecureHook("FriendsList_Update", "ShowFriends")
	self:SecureHook("IgnoreList_Update", "ShowIgnore")
	self:SecureHook("GuildStatus_Update", "ShowGuild")
	self:SecureHook(GameTooltip, "SetUnit", "ScanTooltip")
	self:Hook("GetWhoInfo", "ScanWho", true)
	self:SecureHook("SendWho", function(filter) self:ListenFor("Who") end)
	--self:Hook("SetWhoToUI", function(state) self:Print("SetWhoToUI", state); self.hooks["SetWhoToUI"](state) end)
end

function AuldLangSyne:CHAT_MSG_SYSTEM(message)
	self:Debug("CHAT_MSG_SYSTEM", message)
	if self.listeningForLogin then
		local name = deformat(message, ERR_FRIEND_ONLINE_SS)
		if name and self.db.realm.chars[name] and self.db.realm.chars[name].note then
			--We schedule an event, so we can print the note _after_ the "[name] has come online" message.
			self:ScheduleEvent(self.name.."PrintNote", function()
					self:Print(name .. ": " .. self.db.realm.chars[name].note)
				end, 0.1)
		end
	end
	if self.listeningForWho then
		--local name = deformat(message, WHO_LIST_GUILD_FORMAT) or deformat(message, WHO_LIST_FORMAT)
		local _, name, level, race, class, guild, location = deformat(message, WHO_LIST_GUILD_FORMAT)
		if not name then _, name, level, race, class, location = deformat(message, WHO_LIST_GUILD_FORMAT) end
		if name and self.db.realm.chars[name] ~= nil then
			self:Debug("We know", name)
			if guild then self.db.realm.chars[name].guild = guild end
			self.db.realm.chars[name].level = level
			self.db.realm.chars[name].race = race
			self.db.realm.chars[name].class = class
			self.db.realm.chars[name].updated = time()
			if self.db.profile.show.onwho and self.db.realm.chars[name].note then
				self:ScheduleEvent(self.name.."PrintNote", function()
						self:Print(name .. ": " .. self.db.realm.chars[name].note)
					end, 0.1)
			end
		end
	end
end

function AuldLangSyne:ListenFor(which, stop)
	if stop then
		self["listeningFor"..which] = false
		if self:IsEventRegistered("CHAT_MSG_SYSTEM") then
			self:UnregisterEvent("CHAT_MSG_SYSTEM")
		end
	else
		self["listeningFor"..which] = true
		self:RegisterEvent("CHAT_MSG_SYSTEM")
		self:ScheduleEvent(self.ListenFor, 1, self, which, true)
	end
end

function AuldLangSyne:ShowFriends()
	self:Debug("ShowFriends")
	if self.db.profile.show.onlogon then
		--This might be a friend-has-signed-on update.  So:
		--(Note that this is why we're not just using a bucket event or throttled event.)
		self:ListenFor("Login")
	end
	if FriendsFrame:IsVisible() then
		local friendOffset = FauxScrollFrame_GetOffset(FriendsFrameFriendsScrollFrame);
		local numFriends = GetNumFriends();
		local n = FRIENDS_TO_DISPLAY

		if numFriends < FRIENDS_TO_DISPLAY then n = numFriends end

		for i=1, n, 1 do
			local name, level, class, area, connected, status = GetFriendInfo(i + friendOffset);

			-- They need a note button.
			self.friendbuttons[i].name = name
			if self.db.realm.chars[name] == nil or self.db.realm.chars[name].note == "" or self.db.realm.chars[name].note == nil then
				self.friendbuttons[i]:GetNormalTexture():SetVertexColor(0.5,0.5,0.5)
			else
				self.friendbuttons[i]:GetNormalTexture():SetVertexColor(1,1,1)
			end

			-- Do we know anything about them?
			if self.db.realm.chars[name] and self.db.realm.chars[name].level then
				self:Debug("Munging "..name)
				local nameLocationText = getglobal("FriendsFrameFriendButton"..(i).."ButtonTextNameLocation");
				local infoText = getglobal("FriendsFrameFriendButton"..(i).."ButtonTextInfo");

				local race = self.db.realm.chars[name].race and (self.db.realm.chars[name].race.." ") or ""
				local rank = (self.db.realm.chars[name].rank and self.db.realm.chars[name].rank ~= 0) and ((self.db.profile.show.pvpnum and ("[R".. tonumber(self.db.realm.chars[name].rank)-4 .."]") or self:GetRankName(self.db.realm.chars[name].rank, self.db.realm.chars[name].sex)).." ") or ""
				local lastseen = ""
				if not connected then
					level = self.db.realm.chars[name].level
					class = self.db.realm.chars[name].class
					lastseen = connected and nil or (self.db.realm.chars[name].updated and math.ceil((time() - self.db.realm.chars[name].updated)/86400))
					if connected and nil or self.db.realm.chars[name].updated then
						local currentTime = time()
						local minutes = math.ceil((currentTime - self.db.realm.chars[name].updated) / 60)
						if minutes > 59 then
							local hours = math.ceil((currentTime - self.db.realm.chars[name].updated) / 3600)
							if hours > 23 then
								local days = math.ceil((currentTime - self.db.realm.chars[name].updated) / 86400)
								lastseen = days..L[" day"]..((days > 1) and L["s"] or "")
							else
								lastseen = hours..L[" hour"]..((hours > 1) and L["s"] or "")
							end
						else
							lastseen = minutes..L[" minute"]..((minutes > 1) and L["s"] or "")
						end
					end
				end

				if self.db.profile.show.compact then
					-- Right now offline friends that have never been seen by the addon look out of place, for they are not getting munged.  Fix?
					local pvprank = self.db.realm.chars[name].rank
					if not pvprank then
						pvprank = "?"
					else
						pvprank = tonumber(pvprank)-4
					end
					local nlt = (connected and "" or "|cff999999")..(self.db.profile.show.color.class and ("|cff"..BC:GetHexColor(class)) or "")..name..(self.db.profile.show.color.class and "|r" or "")..((self.db.profile.show.color.name and self.db.realm.chars[name].sex) and "|r" or "")..(connected and "" or "|cff999999")..(self.db.profile.show.pvp and (" / R"..pvprank) or "")..(self.db.profile.show.clevels and (" / L"..self.db.realm.chars[name].level) or "")..((self.db.profile.show.guild and self.db.realm.chars[name].guild) and (" / "..self.db.realm.chars[name].guild) or "")..(connected and (" / "..area..((status ~= "") and " ("..status..")" or "")) or (lastseen and (" / "..lastseen)))
					local it = "|cff999999"..((self.db.realm.chars[name].note and (self.db.realm.chars[name].note ~= "")) and self.db.realm.chars[name].note or "")

					nameLocationText:SetText(nlt)
					infoText:SetText(it)
				else
					local nlt = ((self.db.profile.show.color.name and self.db.realm.chars[name].sex) and ("|cff"..self.colors_sex[self.db.realm.chars[name].sex]) or "")..(self.db.profile.show.pvp and rank or "")..name..((self.db.profile.show.color.name and self.db.realm.chars[name].sex) and "|r" or "")..(connected and "" or ("|cff999999 - "..lastseen..L[" ago"]))
					local it = race..(self.db.profile.show.color.class and ("|cff"..BC:GetHexColor(class)) or "")..class..(self.db.profile.show.color.class and "|r" or "")..(connected and "" or "|cff999999")..((self.db.profile.show.guild and self.db.realm.chars[name].guild) and (" <"..self.db.realm.chars[name].guild..">") or "")

					if connected then
						nameLocationText:SetText(format(FRIENDS_LIST_TEMPLATE, nlt, area, status))
					else
						nameLocationText:SetText(format(FRIENDS_LIST_OFFLINE_TEMPLATE, nlt))
					end
					infoText:SetText(format(FRIENDS_LEVEL_TEMPLATE, level, it))
				end
				-- Make sure we're not overflowing
				if nameLocationText:GetWidth() > 275 then
					nameLocationText:SetJustifyH("LEFT")
					nameLocationText:SetWidth(275)
					nameLocationText:SetHeight(14) -- If we don't set the height, the line wraps.
				end
				if infoText:GetWidth() > 275 then
					infoText:SetJustifyH("LEFT")
					infoText:SetWidth(275)
					infoText:SetHeight(14) -- If we don't set the height, the line wraps.
				end
			end
		end
	end
end

function AuldLangSyne:UpdateFriends()
	local numFriends = GetNumFriends()

	for i=1, numFriends do
		local name, level, class, area, connected, status = GetFriendInfo(i)
		self:Debug(name, level, class, area, connected, status)

		if name and connected then
			if self.db.realm.chars[name] == nil then self.db.realm.chars[name] = {} end
			self.db.realm.chars[name].level = level
			self.db.realm.chars[name].class = class
			self.db.realm.chars[name].updated = time()
		end
	end

	self:Debug("Friends: scanned")
end

function AuldLangSyne:ShowIgnore()
	if FriendsFrame:IsVisible() then
		local ignoreOffset = FauxScrollFrame_GetOffset(FriendsFrameIgnoreScrollFrame);
		for i=1, IGNORES_TO_DISPLAY, 1 do
			local ignoreIndex = i + ignoreOffset;
			local name = GetIgnoreName(ignoreIndex);
			self.ignorebuttons[i].name = name
			if self.db.realm.ignore[name] == nil or self.db.realm.ignore[name] == "" then
				self.ignorebuttons[i]:GetNormalTexture():SetVertexColor(0.5,0.5,0.5)
			else
				self.ignorebuttons[i]:GetNormalTexture():SetVertexColor(1,1,1)
			end
		end
	end
end

function AuldLangSyne:ShowGuild()
	if FriendsFrame:IsVisible() then
		local guildOffset = FauxScrollFrame_GetOffset(GuildListScrollFrame);
		for i=1, GUILDMEMBERS_TO_DISPLAY, 1 do
			local guildIndex = i + guildOffset;
			local name = GetGuildRosterInfo(guildIndex);
			self.guildbuttons[i].name = name
			if self.db.realm.guild[name] == nil or self.db.realm.guild[name] == "" then
				self.guildbuttons[i]:GetNormalTexture():SetVertexColor(0.5,0.5,0.5)
			else
				self.guildbuttons[i]:GetNormalTexture():SetVertexColor(1,1,1)
			end
		end
	end
end

function AuldLangSyne:ScanTooltip(object, unitid)
	self:Debug(unitid)

	if UnitIsPlayer(unitid) then
		local name = UnitName(unitid)
		if self.db.realm.chars[name] ~= nil then
			self:Debug("We know",name)
			self.db.realm.chars[name].rank = UnitPVPRank(unitid)
			self.db.realm.chars[name].race = UnitRace(unitid)

			local sex = UnitSex(unitid)
			-- 1: neuter/unknown, 2: male, 3: female
			-- We're not storing the output of UnitSex directly, for Blizzard has been known to change it.
			if sex == 2 then
				self.db.realm.chars[name].sex = MALE
			elseif sex == 3 then
				self.db.realm.chars[name].sex = FEMALE
			end

			local guildName, guildRankName, guildRankIndex = GetGuildInfo(unitid)
			self.db.realm.chars[name].guild = guildName
			self.db.realm.chars[name].guildRank = guildRankName

			self.db.realm.chars[name].updated = time()

			if FriendsFrame:IsVisible() and FriendsFrame.selectedTab == 1 then
				FriendsList_Update()
			end
		end
		if self.db.profile.show.intooltip and self:CanMessWithGameTooltip() then
			if self.db.realm.chars[name] and self.db.realm.chars[name].note then
				self:AddToGameTooltipAndFixHeight(L["Friend: "] .. self.db.realm.chars[name].note, 0.5, 0.5, 0.5)
			end
			if self.db.realm.guild[name] then
				self:AddToGameTooltipAndFixHeight(L["Guild: "] .. self.db.realm.guild[name], 0.5, 0.5, 0.5)
			end
			if self.db.realm.ignore[name] then
				self:AddToGameTooltipAndFixHeight(L["Ignore: "] .. self.db.realm.ignore[name], 0.5, 0.5, 0.5)
			end
		end
	end
end

function AuldLangSyne:ScanWho(i)
	self:Debug("ScanWho", i)
	local name, guild, level, race, class, zone, group = self.hooks["GetWhoInfo"](i)
	self:Debug(name, guild, level, race, class, zone, group)
	if self.db.realm.chars[name] ~= nil then
		self:Debug("We know", name)
		if guild ~= "" then self.db.realm.chars[name].guild = guild end
		self.db.realm.chars[name].level = level
		self.db.realm.chars[name].race = race
		self.db.realm.chars[name].class = class
		self.db.realm.chars[name].updated = time()

		if FriendsFrame:IsVisible() and FriendsFrame.selectedTab == 1 then
			FriendsList_Update()
		end
	end

	return name, guild, level, race, class, zone, group
end

function AuldLangSyne:CanMessWithGameTooltip()
	if GameTooltip and getglobal("GameTooltipTextLeft1"):IsVisible() and getglobal("GameTooltipTextLeft1"):GetText() ~= nil then
		return true
	else
		return false
	end
end

function AuldLangSyne:AddToGameTooltipAndFixHeight(text, r, g, b)
	GameTooltip:AddLine(text, r, g, b)

	-- Fix up the width of the tooltip
	local length = getglobal(GameTooltip:GetName() .. "TextLeft" .. GameTooltip:NumLines()):GetStringWidth()
	-- Space for right-border:
	length = length + 22

	GameTooltip:SetHeight(GameTooltip:GetHeight() + 14)
	if length > GameTooltip:GetWidth() then
		GameTooltip:SetWidth(length)
	end
end

function AuldLangSyne:DumpData()
	self:Debug("Printing friends")
	for name, info in pairs(self.db.realm.chars) do
		self:Print((name or info.pvp) .. " [" .. (info.level or "?") .. "]" .. (info.guild and (" <"..info.guild..">") or "") .. ", a " .. (info.race and (info.race.." ") or "") .. (info.class or UNKNOWN) .. " last seen on " .. date(nil, info.updated))
	end
end

function AuldLangSyne:ClearData()
	self.db.realm.chars = {}
end

function AuldLangSyne:CTImport()
	--CTPlayerNotes is popular.  Ergo, we must steal its data.
	--NOTE: This will overwrite any existing notes.  This might be bad.

	if CT_PlayerNotes then
		--We can't scan over the CT_PlayerNotes, as that seems to include listings for all realms.  Durnit.
		for i=1, GetNumFriends() do
			local name, level, class, area, connected, status = GetFriendInfo(i)
			if CT_PlayerNotes[name] then
				if not self.db.realm.chars[name] then self.db.realm.chars[name] = {} end
				self.db.realm.chars[name].note = CT_PlayerNotes[name]
			end
		end
	end
	if CT_GuildNotes then
		for i=1, GetNumGuildMembers(true) do
			local name = GetGuildRosterInfo(i)
			if CT_GuildNotes[name] then
				self.db.realm.guild[name] = CT_GuildNotes[name]
			end
		end
	end
	if CT_IgnoreNotes then
		for i=1, GetNumIgnores() do
			local name = GetIgnoreName(i)
			if CT_IgnoreNotes[name] then
				self.db.realm.ignore[name] = CT_IgnoreNotes[name]
			end
		end
	end
end

function AuldLangSyne:CreateButton(name, parent, point, relativePoint, x, y)
	local button = CreateFrame("Button", name, parent)
	button:SetWidth(16); button:SetHeight(16)

	--Left edge aligned to right edge of parent.
	button:SetPoint(point, parent, relativePoint, x, y)

	button:SetNormalTexture("Interface\\Buttons\\UI-GuildButton-PublicNote-Up")
	button:SetDisabledTexture("Interface\\Buttons\\UI-GuildButton-PublicNote-Disabled")
	button:SetHighlightTexture("Interface\\Buttons\\UI-GuildButton-PublicNote-Up")

	button:SetScript("OnClick", function() self:ButtonClick(this) end)
	button:SetScript("OnEnter", function() self:ButtonEnter(this) end)
	button:SetScript("OnLeave", function() self:ButtonLeave(this) end)

	self:Debug("Created "..name.." (parent: "..parent:GetName()..")")
	return button
end

function AuldLangSyne:ButtonClick(button)
	self:Debug(button:GetName(), button.name)
	if self.editFrame:IsVisible() and self.editFrame.name == button.name then
		self.editFrame.name = ""
		self.editFrame.type = ""
		self.editFrame:Hide()
	else
		self.editFrame.name = button.name
		self.editFrame.type = button.type

		local note
		if button.type == "friend" then
			note = self.db.realm.chars[button.name] and self.db.realm.chars[button.name].note or ""
		elseif button.type == "ignore" then
			note = self.db.realm.ignore[button.name] or ""
		elseif button.type == "guild" then
			note = self.db.realm.guild[button.name] or ""
		end
		self.editFrame.editbox:SetText(note)
		self.editFrame.edit_text:SetText(format(L["Editing note for %s"],button.name))
		self.editFrame:Show()
	end
end

function AuldLangSyne:ButtonEnter(button)
	self:Debug(button.name, button.type)
	local note
	if button.type == "friend" then
		note = self.db.realm.chars[button.name] and self.db.realm.chars[button.name].note or ""
	elseif button.type == "ignore" then
		note = self.db.realm.ignore[button.name] or ""
	elseif button.type == "guild" then
		note = self.db.realm.guild[button.name] or ""
	end
	GameTooltip:SetOwner(button, "ANCHOR_RIGHT")
	GameTooltip:ClearLines()
	GameTooltip:AddLine(L["Click to edit"], 1, 0.7, 0)
	GameTooltip:AddLine(note, 0.6, 0.6, 0.6)
	GameTooltip:Show()
end

function AuldLangSyne:ButtonLeave(button)
	GameTooltip:Hide()
end

function AuldLangSyne:CreateEditFrame()
	local f = CreateFrame("Frame", "AuldLangSyne_Edit", UIParent)
	f:SetFrameStrata("DIALOG")
	f:SetToplevel(true)
	f:SetWidth(300)
	f:SetHeight(100)
	f:SetPoint("CENTER", UIParent) --Center it on the screen.
	f:SetBackdrop({bgFile="Interface\\DialogFrame\\UI-DialogBox-Background", edgeFile="Interface\\DialogFrame\\UI-DialogBox-Border",
		tile=true, tileSize=32, edgeSize=32, insets={left=11, right=12, top=12, bottom=11}})

	local h = f:CreateTexture()
	h:SetTexture("Interface\\DialogFrame\\UI-DialogBox-Header")
	h:SetWidth(256)
	h:SetHeight(64)
	h:SetPoint("TOP", f, "TOP", 0, 12)

	local h_text = f:CreateFontString("AuldLangSyne_EditHeaderText", nil, "GameFontNormal")
	h_text:SetPoint("TOP", h, "TOP", 0, -14)
	h_text:SetText(L["Edit note"])

	local edit_text = f:CreateFontString("AuldLangSyne_EditDescText", nil, "GameFontNormal")
	edit_text:SetPoint("CENTER", f, "CENTER", 0, 20)
	--edit_text:SetText(

	--Somewhere to type.
	local edit = CreateFrame("EditBox", nil, f)
	edit:SetFontObject(ChatFontNormal)
	edit:SetHistoryLines(1); edit:SetMaxLetters(250); edit:SetTextInsets(10,10,0,0)
	edit:SetWidth(250); edit:SetHeight(32)
	edit:SetPoint("CENTER", f)
	edit:SetScript("OnShow", function() this:SetFocus() end)
	edit:SetScript("OnEnterPressed", function() self:SaveNote(self.editFrame.name, this:GetText(), self.editFrame.type) end)
	edit:SetScript("OnEscapePressed", function() this:SetText(""); this:GetParent():Hide(); end)

	--Textures for the editbox border.  ("BACKGROUND"?)
	local edit_l, edit_r, edit_b = edit:CreateTexture(), edit:CreateTexture(), edit:CreateTexture()
	edit_l:SetTexture("Interface\\ChatFrame\\UI-ChatInputBorder-Left"); edit_l:SetWidth(65); edit_l:SetHeight(32); edit_l:SetPoint("LEFT", edit, nil, -10); edit_l:SetTexCoord(0,0.2539,0,1);
	edit_r:SetTexture("Interface\\ChatFrame\\UI-ChatInputBorder-Right"); edit_r:SetWidth(25); edit_r:SetHeight(32); edit_r:SetPoint("RIGHT", edit, nil, 10); edit_r:SetTexCoord(0.9,1,0,1);
	edit_b:SetTexture("Interface\\ChatFrame\\UI-ChatInputBorder-Left"); edit_b:SetWidth(5); edit_b:SetHeight(32); edit_b:SetPoint("LEFT", edit_l, "RIGHT"); edit_b:SetPoint("RIGHT", edit_r, "LEFT"); edit_b:SetTexCoord(0.29296875,1,0,1);

	local b_accept = CreateFrame("Button", nil, f, "GameMenuButtonTemplate")
	b_accept:SetWidth(70); b_accept:SetHeight(21)
	b_accept:SetPoint("BOTTOM", f, "BOTTOM", -42, 12)
	b_accept:SetScript("OnClick", function() self:SaveNote(self.editFrame.name, this:GetParent().editbox:GetText(), self.editFrame.type) end)
	b_accept:SetText(L["Confirm"])

	local b_cancel = CreateFrame("Button", nil, f, "GameMenuButtonTemplate")
	b_cancel:SetWidth(70); b_cancel:SetHeight(21)
	b_cancel:SetPoint("BOTTOM", f, "BOTTOM", 42, 12)
	b_cancel:SetScript("OnClick", function() this:GetParent().editbox:SetText(""); this:GetParent():Hide(); end)
	b_cancel:SetText(L["Cancel"])

	f.editbox = edit
	f.edit_text = edit_text
	f:Hide()
	return f
end

function AuldLangSyne:SaveNote(name, note, ntype)
	--This puts a note into the db, and closes the edit frame if it"s open.
	if note == "" then note = nil end
	if ntype == "friend" then
		if not self.db.realm.chars[name] then self.db.realm.chars[name] = {} end
		self.db.realm.chars[name].note = note
		FriendsList_Update()
	elseif ntype == "ignore" then
		self.db.realm.ignore[name] = note
		IgnoreList_Update()
	elseif ntype == "guild" then
		self.db.realm.guild[name] = note
		GuildStatus_Update()
	end

	if self.editFrame:IsVisible() then
		self.editFrame.editbox:SetText("")
		self.editFrame:Hide()
	end
end

function AuldLangSyne:SaveFriends(slotnum)
	local slot
	if slotnum == "undo" then
		self.undo = {}
		slot = self.undo
	else
		self.db.char.friends[slotnum] = {}
		slot = self.db.char.friends[slotnum]
	end
	for i=1, GetNumFriends() do
		local name, level, class, area, connected, status = GetFriendInfo(i)
		table.insert(slot, name)
	end
	self:Print(format(L["Backed up %d friends to slot %d"], GetNumFriends(),slotnum))
end

function AuldLangSyne:LoadFriends(slotnum)
	local slot
	if slotnum == "undo" then
		slot = self.undo
	else
		slot = self.db.char.friends[slotnum]
	end
	if table.getn(slot) > 0 then
		for i=1, GetNumFriends() do
			RemoveFriend(i)
		end
		for _,name in pairs(slot) do
			AddFriend(name)
		end
	else
		self:Print(L["This slot is currently empty."])
	end
end

function AuldLangSyne:GetRankName(rank, sex)
	--"rank" should be the number of the PVP rank (starts at 5, goes to 19).
	--"sex" should be either MALE or FEMALE.
	return getglobal("PVP_RANK_"..rank.."_"..self.faction..((sex == FEMALE) and "_FEMALE" or ""))
end
```

## AuldLangSyne.toc
```toc
## Interface: 11200
## Title: AuldLangSyne |cff7fff7f -Ace2-|r
## Notes: Should auld acquaintance be forgot, you'll need something to remember them by.
## Notes-esES: Te permite anyadir notas personalizadas en tu lista de amigos.
## Version: 1.1.20556
## Author: Kemayo
## X-Website: https://github.com/Road-block/Select_AddOns
## X-Category: Interface Enhancements
## X-Date: $Date: 2018-06-04 12:21:01 -0500 (Mon, 04 Jun 2018) $
## X-Credits: Roadblock: Fubar and minor fixes for private servers.
## OptionalDeps: Ace2, Babble-2.2, FuBar, FuBarPlugin-2.0, Deformat, TabletLib, DewdropLib
## SavedVariables: AuldLangSyneDB

lib\AceLibrary\AceLibrary.lua
lib\AceOO\AceOO-2.0.lua
lib\AceConsole\AceConsole-2.0.lua
lib\AceLocale\AceLocale-2.2.lua
lib\AceEvent\AceEvent-2.0.lua
lib\AceAddon\AceAddon-2.0.lua
lib\AceHook\AceHook-2.1.lua
lib\AceDB\AceDB-2.0.lua
lib\AceDebug\AceDebug-2.0.lua
lib\Babble-Class-2.2\Babble-Class-2.2.lua
lib\Deformat-2.0\Deformat-2.0.lua
lib\Dewdrop-2.0\Dewdrop-2.0.lua
lib\Tablet-2.0\Tablet-2.0.lua
lib\FuBarPlugin-2.0\FuBarPlugin-2.0.lua

locales\enUS.lua
locales\zhCN.lua
locales\koKR.lua
locales\esES.lua

AuldLangSyne.lua

```

## lib/AceAddon/AceAddon-2.0.lua
```lua
--[[
Name: AceAddon-2.0
Revision: $Rev: 14832 $
Developed by: The Ace Development Team (http://www.wowace.com/index.php/The_Ace_Development_Team)
Inspired By: Ace 1.x by Turan (turan@gryphon.com)
Website: http://www.wowace.com/
Documentation: http://www.wowace.com/index.php/AceAddon-2.0
SVN: http://svn.wowace.com/root/trunk/Ace2/AceAddon-2.0
Description: Base for all Ace addons to inherit from.
Dependencies: AceLibrary, AceOO-2.0, AceEvent-2.0, (optional) AceConsole-2.0
]]

local MAJOR_VERSION = "AceAddon-2.0"
local MINOR_VERSION = "$Revision: 14832 $"

-- This ensures the code is only executed if the libary doesn't already exist, or is a newer version
if not AceLibrary then error(MAJOR_VERSION .. " requires AceLibrary.") end
if not AceLibrary:IsNewVersion(MAJOR_VERSION, MINOR_VERSION) then return end

if not AceLibrary:HasInstance("AceOO-2.0") then error(MAJOR_VERSION .. " requires AceOO-2.0.") end

-- Localization
local STANDBY, TITLE, NOTES, VERSION, AUTHOR, DATE, CATEGORY, EMAIL, WEBSITE, CATEGORIES, ABOUT, PRINT_ADDON_INFO
if GetLocale() == "deDE" then
	STANDBY = "|cffff5050(Standby)|r" -- capitalized

	TITLE = "Titel"
	NOTES = "Anmerkung"
	VERSION = "Version"
	AUTHOR = "Autor"
	DATE = "Datum"
	CATEGORY = "Kategorie"
	EMAIL = "E-mail"
	WEBSITE = "Webseite"
		 
	ABOUT = "\195\188ber"
	PRINT_ADDON_INFO = "Gibt Addondaten aus"

	CATEGORIES = {
		["Action Bars"] = "Aktionsleisten",
		["Auction"] = "Auktion",
		["Audio"] = "Audio",
		["Battlegrounds/PvP"] = "Schlachtfeld/PvP",
		["Buffs"] = "Buffs",
		["Chat/Communication"] = "Chat/Kommunikation",
		["Druid"] = "Druide",
		["Hunter"] = "J�ger",
		["Mage"] = "Magier",
		["Paladin"] = "Paladin",
		["Priest"] = "Priester",
		["Rogue"] = "Schurke",
		["Shaman"] = "Schamane",
		["Warlock"] = "Hexenmeister",
		["Warrior"] = "Krieger",
		["Healer"] = "Heiler",
		["Tank"] = "Tank", -- noone use "Brecher"...
		["Caster"] = "Caster",
		["Combat"] = "Kampf",
		["Compilations"] = "Compilations", -- whats that o_O
		["Data Export"] = "Datenexport",
		["Development Tools"] = "Entwicklungs Tools",
		["Guild"] = "Gilde",
		["Frame Modification"] = "Frame Modifikation",
		["Interface Enhancements"] = "Interface Verbesserungen",
		["Inventory"] = "Inventar",
		["Library"] = "Library",
		["Map"] = "Map",
		["Mail"] = "Mail",
		["Miscellaneous"] = "Diverses",
		["Quest"] = "Quest",
		["Raid"] = "Schlachtzug",
		["Tradeskill"] = "Handelsf\195\164higkeit",
		["UnitFrame"] = "UnitFrame",
	}
elseif GetLocale() == "frFR" then
	STANDBY = "|cffff5050(attente)|r"
	
	TITLE = "Titre"
	NOTES = "Notes"
	VERSION = "Version"
	AUTHOR = "Auteur"
	DATE = "Date"
	CATEGORY = "Cat\195\169gorie"
	EMAIL = "E-mail"
	WEBSITE = "Site web"
	
	ABOUT = "A propos"
	PRINT_ADDON_INFO = "Afficher les informations sur l'addon"
	
	CATEGORIES = {
		["Action Bars"] = "Barres d'action",
		["Auction"] = "H\195\180tel des ventes",
		["Audio"] = "Audio",
		["Battlegrounds/PvP"] = "Champs de bataille/JcJ",
		["Buffs"] = "Buffs",
		["Chat/Communication"] = "Chat/Communication",
		["Druid"] = "Druide",
		["Hunter"] = "Chasseur",
		["Mage"] = "Mage",
		["Paladin"] = "Paladin",
		["Priest"] = "Pr\195\170tre",
		["Rogue"] = "Voleur",
		["Shaman"] = "Chaman",
		["Warlock"] = "D\195\169moniste",
		["Warrior"] = "Guerrier",
		["Healer"] = "Soigneur",
		["Tank"] = "Tank",
		["Caster"] = "Casteur",
		["Combat"] = "Combat",
		["Compilations"] = "Compilations",
		["Data Export"] = "Exportation de donn\195\169es",
		["Development Tools"] = "Outils de d\195\169veloppement",
		["Guild"] = "Guilde",
		["Frame Modification"] = "Modification des fen\195\170tres",
		["Interface Enhancements"] = "Am\195\169liorations de l'interface",
		["Inventory"] = "Inventaire",
		["Library"] = "Biblioth\195\168ques",
		["Map"] = "Carte",
		["Mail"] = "Courrier",
		["Miscellaneous"] = "Divers",
		["Quest"] = "Qu\195\170tes",
		["Raid"] = "Raid",
		["Tradeskill"] = "M\195\169tiers",
		["UnitFrame"] = "Fen\195\170tres d'unit\195\169",
	}
elseif GetLocale() == "koKR" then
	STANDBY = "|cffff5050(사용가능)|r"
	
	TITLE = "제목"
	NOTES = "노트"
	VERSION = "버전"
	AUTHOR = "저작자"
	DATE = "날짜"
	CATEGORY = "분류"
	EMAIL = "E-mail"
	WEBSITE = "웹사이트"
	
	ABOUT = "정보"
	PRINT_ADDON_INFO = "애드온 정보 출력"
	
	CATEGORIES = {
		["Action Bars"] = "액션바",
		["Auction"] = "경매",
		["Audio"] = "음향",
		["Battlegrounds/PvP"] = "전장/PvP",
		["Buffs"] = "버프",
		["Chat/Communication"] = "대화/의사소통",
		["Druid"] = "드루이드",
		["Hunter"] = "사냥꾼",
		["Mage"] = "마법사",
		["Paladin"] = "성기사",
		["Priest"] = "사제",
		["Rogue"] = "도적",
		["Shaman"] = "주술사",
		["Warlock"] = "흑마법사",
		["Warrior"] = "전사",
		["Healer"] = "힐러",
		["Tank"] = "탱커",
		["Caster"] = "캐스터",
		["Combat"] = "전투",
		["Compilations"] = "복합",
		["Data Export"] = "자료 출력",
		["Development Tools"] = "개발 도구",
		["Guild"] = "길드",
		["Frame Modification"] = "구조 변경",
		["Interface Enhancements"] = "인터페이스 강화",
		["Inventory"] = "인벤토리",
		["Library"] = "라이브러리",
		["Map"] = "지도",
		["Mail"] = "우편",
		["Miscellaneous"] = "기타",
		["Quest"] = "퀘스트",
		["Raid"] = "공격대",
		["Tradeskill"] = "전문기술",
		["UnitFrame"] = "유닛 프레임",
	}
elseif GetLocale() == "zhTW" then
	STANDBY = "|cffff5050(待命)|r"
	
	TITLE = "標題"
	NOTES = "註記"
	VERSION = "版本"
	AUTHOR = "作者"
	DATE = "日期"
	CATEGORY = "類別"
	EMAIL = "E-mail"
	WEBSITE = "網站"
	
	ABOUT = "關於"
	PRINT_ADDON_INFO = "顯示插件資訊"
	
	CATEGORIES = {
		["Action Bars"] = "動作列",
		["Auction"] = "拍賣",
		["Audio"] = "音樂",
		["Battlegrounds/PvP"] = "戰場/PvP",
		["Buffs"] = "增益",
		["Chat/Communication"] = "聊天/通訊",
		["Druid"] = "德魯伊",
		["Hunter"] = "獵人",
		["Mage"] = "法師",
		["Paladin"] = "聖騎士",
		["Priest"] = "牧師",
		["Rogue"] = "盜賊",
		["Shaman"] = "薩滿",
		["Warlock"] = "術士",
		["Warrior"] = "戰士",
		["Healer"] = "治療者",
		["Tank"] = "坦克",
		["Caster"] = "施法者",
		["Combat"] = "戰鬥",
		["Compilations"] = "編輯",
		["Data Export"] = "資料匯出",
		["Development Tools"] = "開發工具",
		["Guild"] = "公會",
		["Frame Modification"] = "框架修改",
		["Interface Enhancements"] = "介面增強",
		["Inventory"] = "背包",
		["Library"] = "資料庫",
		["Map"] = "地圖",
		["Mail"] = "郵件",
		["Miscellaneous"] = "綜合",
		["Quest"] = "任務",
		["Raid"] = "團隊",
		["Tradeskill"] = "商業技能",
		["UnitFrame"] = "單位框架",
	}
elseif GetLocale() == "zhCN" then
	STANDBY = "|cffff5050(\230\154\130\230\140\130)|r"
	
	TITLE = "\230\160\135\233\162\152"
	NOTES = "\233\153\132\230\179\168"
	VERSION = "\231\137\136\230\156\172"
	AUTHOR = "\228\189\156\232\128\133"
	DATE = "\230\151\165\230\156\159"
	CATEGORY = "\229\136\134\231\177\187"
	EMAIL = "\231\148\181\229\173\144\233\130\174\228\187\182"
	WEBSITE = "\231\189\145\231\171\153"
	
	ABOUT = "\229\133\179\228\186\142"
	PRINT_ADDON_INFO = "\229\141\176\229\136\151\229\135\186\230\143\146\228\187\182\228\191\161\230\129\175"
	
	CATEGORIES = {
		["Action Bars"] = "\229\138\168\228\189\156\230\157\161",
		["Auction"] = "\230\139\141\229\141\150",
		["Audio"] = "\233\159\179\233\162\145",
		["Battlegrounds/PvP"] = "\230\136\152\229\156\186/PvP",
		["Buffs"] = "\229\162\158\231\155\138\233\173\148\230\179\149",
		["Chat/Communication"] = "\232\129\138\229\164\169/\228\186\164\230\181\129",
		["Druid"] = "\229\190\183\233\178\129\228\188\138",
		["Hunter"] = "\231\140\142\228\186\186",
		["Mage"] = "\230\179\149\229\184\136",
		["Paladin"] = "\229\156\163\233\170\145\229\163\171",
		["Priest"] = "\231\137\167\229\184\136",
		["Rogue"] = "\231\155\151\232\180\188",
		["Shaman"] = "\232\144\168\230\187\161\231\165\173\229\143\184",
		["Warlock"] = "\230\156\175\229\163\171",
		["Warrior"] = "\230\136\152\229\163\171",
--		["Healer"] = "\230\178\187\231\150\151\228\191\157\233\154\156",
--		["Tank"] = "\232\191\145\230\136\152\230\142\167\229\136\182",
--		["Caster"] = "\232\191\156\231\168\139\232\190\147\229\135\186",
		["Combat"] = "\230\136\152\230\150\151",
		["Compilations"] = "\231\188\150\232\175\145",
		["Data Export"] = "\230\149\176\230\141\174\229\175\188\229\135\186",
		["Development Tools"] = "\229\188\128\229\143\145\229\183\165\229\133\183",
		["Guild"] = "\229\133\172\228\188\154",
		["Frame Modification"] = "\230\161\134\230\158\182\228\191\174\230\148\185",
		["Interface Enhancements"] = "\231\149\140\233\157\162\229\162\158\229\188\186",
		["Inventory"] = "\232\131\140\229\140\133",
		["Library"] = "\229\186\147",
		["Map"] = "\229\156\176\229\155\190",
		["Mail"] = "\233\130\174\228\187\182",
		["Miscellaneous"] = "\230\157\130\233\161\185",
		["Quest"] = "\228\187\187\229\138\161",
		["Raid"] = "\229\155\162\233\152\159",
		["Tradeskill"] = "\229\149\134\228\184\154\230\138\128\232\131\189",
		["UnitFrame"] = "\229\164\180\229\131\143\230\161\134\230\158\182",
	}
else -- enUS
	STANDBY = "|cffff5050(standby)|r"
	
	TITLE = "Title"
	NOTES = "Notes"
	VERSION = "Version"
	AUTHOR = "Author"
	DATE = "Date"
	CATEGORY = "Category"
	EMAIL = "E-mail"
	WEBSITE = "Website"
	
	ABOUT = "About"
	PRINT_ADDON_INFO = "Print out addon info"
	
	CATEGORIES = {
		["Action Bars"] = "Action Bars",
		["Auction"] = "Auction",
		["Audio"] = "Audio",
		["Battlegrounds/PvP"] = "Battlegrounds/PvP",
		["Buffs"] = "Buffs",
		["Chat/Communication"] = "Chat/Communication",
		["Druid"] = "Druid",
		["Hunter"] = "Hunter",
		["Mage"] = "Mage",
		["Paladin"] = "Paladin",
		["Priest"] = "Priest",
		["Rogue"] = "Rogue",
		["Shaman"] = "Shaman",
		["Warlock"] = "Warlock",
		["Warrior"] = "Warrior",
		["Healer"] = "Healer",
		["Tank"] = "Tank",
		["Caster"] = "Caster",
		["Combat"] = "Combat",
		["Compilations"] = "Compilations",
		["Data Export"] = "Data Export",
		["Development Tools"] = "Development Tools",
		["Guild"] = "Guild",
		["Frame Modification"] = "Frame Modification",
		["Interface Enhancements"] = "Interface Enhancements",
		["Inventory"] = "Inventory",
		["Library"] = "Library",
		["Map"] = "Map",
		["Mail"] = "Mail",
		["Miscellaneous"] = "Miscellaneous",
		["Quest"] = "Quest",
		["Raid"] = "Raid",
		["Tradeskill"] = "Tradeskill",
		["UnitFrame"] = "UnitFrame",
	}
end

setmetatable(CATEGORIES, { __index = function(self, key) -- case-insensitive
	local lowerKey = string.lower(key)
	for k,v in CATEGORIES do
		if string.lower(k) == lowerKey then
			return v
		end
	end
end })

-- Create the library object

local AceOO = AceLibrary("AceOO-2.0")
local AceAddon = AceOO.Class()
local AceEvent
local AceConsole
local AceModuleCore

function AceAddon:ToString()
	return "AceAddon"
end

local function print(text)
	DEFAULT_CHAT_FRAME:AddMessage(text)
end

function AceAddon:ADDON_LOADED(name)
	while table.getn(self.nextAddon) > 0 do
		local addon = table.remove(self.nextAddon, 1)
		table.insert(self.addons, addon)
		if not self.addons[name] then
			self.addons[name] = addon
		end
		self:InitializeAddon(addon, name)
	end
end

local function RegisterOnEnable(self)
	if DEFAULT_CHAT_FRAME and DEFAULT_CHAT_FRAME.defaultLanguage then -- HACK
		AceAddon.playerLoginFired = true
	end
	if AceAddon.playerLoginFired then
		AceAddon.addonsStarted[self] = true
		if (type(self.IsActive) ~= "function" or self:IsActive()) and (not AceModuleCore or not AceModuleCore:IsModule(self) or AceModuleCore:IsModuleActive(self)) then
			local current = self.class
			while true do
				if current == AceOO.Class then
					break
				end
				if current.mixins then
					for mixin in pairs(current.mixins) do
						if type(mixin.OnEmbedEnable) == "function" then
							mixin:OnEmbedEnable(self)
						end
					end
				end
				current = current.super
			end
			if type(self.OnEnable) == "function" then
				self:OnEnable()
			end
		end
	else
		if not AceAddon.addonsToOnEnable then
			AceAddon.addonsToOnEnable = {}
		end
		table.insert(AceAddon.addonsToOnEnable, self)
	end
end

local function stripSpaces(text)
	if type(text) == "string" then
		return (string.gsub(string.gsub(text, "^%s*(.-)%s*$", "%1"), "%s%s+", " "))
	end
	return text
end

function AceAddon:InitializeAddon(addon, name)
	if addon.name == nil then
		addon.name = name
	end
	if GetAddOnMetadata then
		-- TOC checks
		if addon.title == nil then
			addon.title = GetAddOnMetadata(name, "Title")
			if addon.title then
				local num = string.find(addon.title, " |cff7fff7f %-Ace2%-|r$")
				if num then
					addon.title = string.sub(addon.title, 1, num - 1)
				end
				addon.title = stripSpaces(addon.title)
			end
		end
		if addon.notes == nil then
			addon.notes = GetAddOnMetadata(name, "Notes")
			addon.notes = stripSpaces(addon.notes)
		end
		if addon.version == nil then
			addon.version = GetAddOnMetadata(name, "Version")
			if addon.version then
				if string.find(addon.version, "%$Revision: (%d+) %$") then
					addon.version = string.gsub(addon.version, "%$Revision: (%d+) %$", "%1")
				elseif string.find(addon.version, "%$Rev: (%d+) %$") then
					addon.version = string.gsub(addon.version, "%$Rev: (%d+) %$", "%1")
				elseif string.find(addon.version, "%$LastChangedRevision: (%d+) %$") then
					addon.version = string.gsub(addon.version, "%$LastChangedRevision: (%d+) %$", "%1")
				end
			end
			addon.version = stripSpaces(addon.version)
		end
		if addon.author == nil then
			addon.author = GetAddOnMetadata(name, "Author")
			addon.author = stripSpaces(addon.author)
		end
		if addon.date == nil then
			addon.date = GetAddOnMetadata(name, "X-Date") or GetAddOnMetadata(name, "X-ReleaseDate")
			if addon.date then
				if string.find(addon.date, "%$Date: (.-) %$") then
					addon.date = string.gsub(addon.date, "%$Date: (.-) %$", "%1")
				elseif string.find(addon.date, "%$LastChangedDate: (.-) %$") then
					addon.date = string.gsub(addon.date, "%$LastChangedDate: (.-) %$", "%1")
				end
			end
			addon.date = stripSpaces(addon.date)
		end
		if addon.category == nil then
			addon.category = GetAddOnMetadata(name, "X-Category")
			addon.category = stripSpaces(addon.category)
		end
		if addon.email == nil then
			addon.email = GetAddOnMetadata(name, "X-eMail") or GetAddOnMetadata(name, "X-Email")
			addon.email = stripSpaces(addon.email)
		end
		if addon.website == nil then
			addon.website = GetAddOnMetadata(name, "X-Website")
			addon.website = stripSpaces(addon.website)
		end
	end
	local current = addon.class
	while true do
		if current == AceOO.Class then
			break
		end
		if current.mixins then
			for mixin in pairs(current.mixins) do
				if type(mixin.OnEmbedInitialize) == "function" then
					mixin:OnEmbedInitialize(addon, name)
				end
			end
		end
		current = current.super
	end
	if type(addon.OnInitialize) == "function" then
		addon:OnInitialize(name)
	end
	RegisterOnEnable(addon)
end

function AceAddon.prototype:PrintAddonInfo()
	local x
	if self.title then
		x = "|cffffff7f" .. tostring(self.title) .. "|r"
	elseif self.name then
		x = "|cffffff7f" .. tostring(self.name) .. "|r"
	else
		x = "|cffffff7f<" .. tostring(self.class) .. " instance>|r"
	end
	if type(self.IsActive) == "function" then
		if not self:IsActive() then
			x = x .. " " .. STANDBY
		end
	end
	if self.version then
		x = x .. " - |cffffff7f" .. tostring(self.version) .. "|r"
	end
	if self.notes then
		x = x .. " - " .. tostring(self.notes)
	end
	print(x)
	if self.author then
		print(" - |cffffff7f" .. AUTHOR .. ":|r " .. tostring(self.author))
	end
	if self.date then
		print(" - |cffffff7f" .. DATE .. ":|r " .. tostring(self.date))
	end
	if self.category then
		local category = CATEGORIES[self.category]
		if category then
			print(" - |cffffff7f" .. CATEGORY .. ":|r " .. category)
		end
	end
	if self.email then
		print(" - |cffffff7f" .. EMAIL .. ":|r " .. tostring(self.email))
	end
	if self.website then
		print(" - |cffffff7f" .. WEBSITE .. ":|r " .. tostring(self.website))
	end
end

local options
function AceAddon:GetAceOptionsDataTable(target)
	if not options then
		options = {
			about = {
				name = ABOUT,
				desc = PRINT_ADDON_INFO,
				type = "execute",
				func = "PrintAddonInfo",
				order = -1,
			}
		}
	end
	return options
end

function AceAddon:PLAYER_LOGIN()
	self.playerLoginFired = true
	if self.addonsToOnEnable then
		while table.getn(self.addonsToOnEnable) > 0 do
			local addon = table.remove(self.addonsToOnEnable, 1)
			self.addonsStarted[addon] = true
			if (type(addon.IsActive) ~= "function" or addon:IsActive()) and (not AceModuleCore or not AceModuleCore:IsModule(addon) or AceModuleCore:IsModuleActive(addon)) then
				local current = addon.class
				while true do
					if current == AceOO.Class then
						break
					end
					if current.mixins then
						for mixin in pairs(current.mixins) do
							if type(mixin.OnEmbedEnable) == "function" then
								mixin:OnEmbedEnable(addon)
							end
						end
					end
					current = current.super
				end
				if type(addon.OnEnable) == "function" then
					addon:OnEnable()
				end
			end
		end
		self.addonsToOnEnable = nil
	end
end

function AceAddon.prototype:Inject(t)
	AceAddon:argCheck(t, 2, "table")
	for k,v in pairs(t) do
		self[k] = v
	end
end

function AceAddon.prototype:init()
	if not AceEvent then
		error(MAJOR_VERSION .. " requires AceEvent-2.0", 4)
	end
	AceAddon.super.prototype.init(self)
	
	self.super = self.class.prototype
	
	AceAddon:RegisterEvent("ADDON_LOADED", "ADDON_LOADED", true)
	table.insert(AceAddon.nextAddon, self)
end

function AceAddon.prototype:ToString()
	local x
	if type(self.title) == "string" then
		x = self.title
	elseif type(self.name) == "string" then
		x = self.name
	else
		x = "<" .. tostring(self.class) .. " instance>"
	end
	if (type(self.IsActive) == "function" and not self:IsActive()) or (AceModuleCore and AceModuleCore:IsModule(addon) and AceModuleCore:IsModuleActive(addon)) then
		x = x .. " " .. STANDBY
	end
	return x
end

AceAddon.new = function(self, m1, m2, m3, m4, m5, m6, m7, m8, m9, m10, m11, m12, m13, m14, m15, m16, m17, m18, m19, m20)
	local class = AceAddon:pcall(AceOO.Classpool, self, m1, m2, m3, m4, m5, m6, m7, m8, m9, m10, m11, m12, m13, m14, m15, m16, m17, m18, m19, m20)
	return class:new()
end

local function external(self, major, instance)
	if major == "AceEvent-2.0" then
		AceEvent = instance
		
		AceEvent:embed(self)
		
		self:RegisterEvent("PLAYER_LOGIN", "PLAYER_LOGIN", true)
	elseif major == "AceConsole-2.0" then
		AceConsole = instance
		
		local slashCommands = { "/ace2" }
		local _,_,_,enabled,loadable = GetAddOnInfo("Ace")
		if not enabled or not loadable then
			table.insert(slashCommands, "/ace")
		end
		local function listAddon(addon, depth)
			if not depth then
				depth = 0
			end
			
			local s = string.rep("  ", depth) .. " - " .. tostring(addon)
			if rawget(addon, 'version') then
				s = s .. " - |cffffff7f" .. tostring(addon.version) .. "|r"
			end
			if rawget(addon, 'slashCommand') then
				s = s .. " |cffffff7f(" .. tostring(addon.slashCommand) .. ")|r"
			end
			print(s)
			if type(rawget(addon, 'modules')) == "table" then
				local i = 0
				for k,v in pairs(addon.modules) do
					i = i + 1
					if i == 6 then
						print(string.rep("  ", depth + 1) .. " - more...")
						break
					else
						listAddon(v, depth + 1)
					end
				end
			end
		end
		local function listNormalAddon(i)
			local name,_,_,enabled,loadable = GetAddOnInfo(i)
			if not loadable then
				enabled = false
			end
			if self.addons[name] then
				local addon = self.addons[name]
				if not AceCoreAddon or not AceCoreAddon:IsModule(addon) then
					listAddon(addon)
				end
			else
				local s = " - " .. tostring(GetAddOnMetadata(i, "Title") or name)
				local version = GetAddOnMetadata(i, "Version")
				if version then
					if string.find(version, "%$Revision: (%d+) %$") then
						version = string.gsub(version, "%$Revision: (%d+) %$", "%1")
					elseif string.find(version, "%$Rev: (%d+) %$") then
						version = string.gsub(version, "%$Rev: (%d+) %$", "%1")
					elseif string.find(version, "%$LastChangedRevision: (%d+) %$") then
						version = string.gsub(version, "%$LastChangedRevision: (%d+) %$", "%1")
					end
					s = s .. " - |cffffff7f" .. version .. "|r"
				end
				if not enabled then
					s = s .. " |cffff0000(disabled)|r"
				end
				if IsAddOnLoadOnDemand(i) then
					s = s .. " |cff00ff00[LoD]|r"
				end
				print(s)
			end
		end
		local function mySort(alpha, bravo)
			return tostring(alpha) < tostring(bravo)
		end
		AceConsole.RegisterChatCommand(self, slashCommands, {
			desc = "AddOn development framework",
			name = "Ace2",
			type = "group",
			args = {
				about = {
					desc = "Get information about Ace2",
					name = "About",
					type = "execute",
					func = function()
						print("|cffffff7fAce2|r - |cffffff7f2.0." .. string.gsub(MINOR_VERSION, "%$Revision: (%d+) %$", "%1") .. "|r - AddOn development framework")
						print(" - |cffffff7f" .. AUTHOR .. ":|r Ace Development Team")
						print(" - |cffffff7f" .. WEBSITE .. ":|r http://www.wowace.com/")
					end
				},
				list = {
					desc = "List addons",
					name = "List",
					type = "group",
					args = {
						ace2 = {
							desc = "List addons using Ace2",
							name = "Ace2",
							type = "execute",
							func = function()
								print("|cffffff7fAddon list:|r")
								local AceCoreAddon = AceLibrary:HasInstance("AceCoreAddon-2.0") and AceLibrary("AceCoreAddon-2.0")
								table.sort(self.addons, mySort)
								for _,v in ipairs(self.addons) do
									if not AceCoreAddon or not AceCoreAddon:IsModule(v) then
										listAddon(v)
									end
								end
							end
						},
						all = {
							desc = "List all addons",
							name = "All",
							type = "execute",
							func = function()
								print("|cffffff7fAddon list:|r")
								local AceCoreAddon = AceLibrary:HasInstance("AceCoreAddon-2.0") and AceLibrary("AceCoreAddon-2.0")
								local count = GetNumAddOns()
								for i = 1, count do
									listNormalAddon(i)
								end
							end
						},
						enabled = {
							desc = "List all enabled addons",
							name = "Enabled",
							type = "execute",
							func = function()
								print("|cffffff7fAddon list:|r")
								local AceCoreAddon = AceLibrary:HasInstance("AceCoreAddon-2.0") and AceLibrary("AceCoreAddon-2.0")
								local count = GetNumAddOns()
								for i = 1, count do
									local _,_,_,enabled,loadable = GetAddOnInfo(i)
									if enabled and loadable then
										listNormalAddon(i)
									end
								end
							end
						},
						disabled = {
							desc = "List all disabled addons",
							name = "Disabled",
							type = "execute",
							func = function()
								print("|cffffff7fAddon list:|r")
								local AceCoreAddon = AceLibrary:HasInstance("AceCoreAddon-2.0") and AceLibrary("AceCoreAddon-2.0")
								local count = GetNumAddOns()
								for i = 1, count do
									local _,_,_,enabled,loadable = GetAddOnInfo(i)
									if not enabled or not loadable then
										listNormalAddon(i)
									end
								end
							end
						},
						lod = {
							desc = "List all LoadOnDemand addons",
							name = "LoadOnDemand",
							type = "execute",
							func = function()
								print("|cffffff7fAddon list:|r")
								local AceCoreAddon = AceLibrary:HasInstance("AceCoreAddon-2.0") and AceLibrary("AceCoreAddon-2.0")
								local count = GetNumAddOns()
								for i = 1, count do
									if IsAddOnLoadOnDemand(i) then
										listNormalAddon(i)
									end
								end
							end
						},
						ace1 = {
							desc = "List all addons using Ace1",
							name = "Ace 1.x",
							type = "execute",
							func = function()
								print("|cffffff7fAddon list:|r")
								local count = GetNumAddOns()
								for i = 1, count do
									local dep1, dep2, dep3, dep4 = GetAddOnDependencies(i)
									if dep1 == "Ace" or dep2 == "Ace" or dep3 == "Ace" or dep4 == "Ace" then
										listNormalAddon(i)
									end
								end
							end
						},
						libs = {
							desc = "List all libraries using AceLibrary",
							name = "Libraries",
							type = "execute",
							func = function()
								if type(AceLibrary) == "table" and type(AceLibrary.libs) == "table" then
									print("|cffffff7fLibrary list:|r")
									for name, data in pairs(AceLibrary.libs) do
										local s
										if data.minor then
											s = " - " .. tostring(name) .. "." .. tostring(data.minor)
										else
											s = " - " .. tostring(name)
										end
										if AceLibrary(name).slashCommand then
											s = s .. " |cffffff7f(" .. tostring(AceLibrary(name).slashCommand) .. "|cffffff7f)"
										end
										print(s)
									end
								end
							end
						},
						search = {
							desc = "Search by name",
							name = "Search",
							type = "text",
							usage = "<keyword>",
							input = true,
							get = false,
							set = function(...)
								for i,v in ipairs(arg) do
									arg[i] = string.lower(string.gsub(string.gsub(v, '%*', '.*'), '%%', '%%%%'))
								end
								local count = GetNumAddOns()
								for i = 1, count do
									local name = GetAddOnInfo(i)
									local good = true
									for _,v in ipairs(arg) do
										if not string.find(string.lower(name), v) then
											good = false
											break
										end
									end
									if good then
										listNormalAddon(i)
									end
								end
							end
						}
					},
				},
				enable = {
					desc = "Enable addon",
					name = "Enable",
					type = "text",
					usage = "<addon>",
					get = false,
					set = function(text)
						local name,title,_,_,_,reason = GetAddOnInfo(text)
						if reason == "MISSING" then
							print(string.format("|cffffff7fAce2:|r AddOn %q does not exist", text))
						else
							EnableAddOn(text)
							print(string.format("|cffffff7fAce2:|r %s is now enabled", title or name))
						end
					end,
				},
				disable = {
					desc = "Disable addon",
					name = "Disable",
					type = "text",
					usage = "<addon>",
					get = false,
					set = function(text)
						local name,title,_,_,_,reason = GetAddOnInfo(text)
						if reason == "MISSING" then
							print(string.format("|cffffff7fAce2:|r AddOn %q does not exist", text))
						else
							DisableAddOn(text)
							print(string.format("|cffffff7fAce2:|r %s is now disabled", title or name))
						end
					end,
				},
				load = {
					desc = "Load addon",
					name = "Load",
					type = "text",
					usage = "<addon>",
					get = false,
					set = function(text)
						local name,title,_,_,loadable,reason = GetAddOnInfo(text)
						if reason == "MISSING" then
							print(string.format("|cffffff7fAce2:|r AddOn %q does not exist.", text))
						elseif not loadable then
							print(string.format("|cffffff7fAce2:|r AddOn %q is not loadable. Reason: %s", text, reason))
						else
							LoadAddOn(text)
							print(string.format("|cffffff7fAce2:|r %s is now loaded", title or name))
						end
					end
				},
				info = {
					desc = "Display information",
					name = "Information",
					type = "execute",
					func = function()
						local mem, threshold = gcinfo()
						print(string.format(" - |cffffff7fMemory usage [|r%.3f MiB|cffffff7f]|r", mem / 1024))
						print(string.format(" - |cffffff7fThreshold [|r%.3f MiB|cffffff7f]|r", threshold / 1024))
						print(string.format(" - |cffffff7fFramerate [|r%.0f fps|cffffff7f]|r", GetFramerate()))
						local bandwidthIn, bandwidthOut, latency = GetNetStats()
						bandwidthIn, bandwidthOut = floor(bandwidthIn * 1024), floor(bandwidthOut * 1024)
						print(string.format(" - |cffffff7fLatency [|r%.0f ms|cffffff7f]|r", latency))
						print(string.format(" - |cffffff7fBandwidth in [|r%.0f B/s|cffffff7f]|r", bandwidthIn))
						print(string.format(" - |cffffff7fBandwidth out [|r%.0f B/s|cffffff7f]|r", bandwidthOut))
						print(string.format(" - |cffffff7fTotal addons [|r%d|cffffff7f]|r", GetNumAddOns()))
						print(string.format(" - |cffffff7fAce2 addons [|r%d|cffffff7f]|r", table.getn(self.addons)))
						local ace = 0
						local enabled = 0
						local disabled = 0
						local lod = 0
						for i = 1, GetNumAddOns() do
							local dep1, dep2, dep3, dep4 = GetAddOnDependencies(i)
							if dep1 == "Ace" or dep2 == "Ace" or dep3 == "Ace" or dep4 == "Ace" then
								ace = ace + 1
							end
							if IsAddOnLoadOnDemand(i) then
								lod = lod + 1
							end
							local _,_,_,IsActive,loadable = GetAddOnInfo(i)
							if not IsActive or not loadable then
								disabled = disabled + 1
							else
								enabled = enabled + 1
							end
						end
						print(string.format(" - |cffffff7fAce 1.x addons [|r%d|cffffff7f]|r", ace))
						print(string.format(" - |cffffff7fLoadOnDemand addons [|r%d|cffffff7f]|r", lod))
						print(string.format(" - |cffffff7fenabled addons [|r%d|cffffff7f]|r", enabled))
						print(string.format(" - |cffffff7fdisabled addons [|r%d|cffffff7f]|r", disabled))
						local libs = 0
						if type(AceLibrary) == "table" and type(AceLibrary.libs) == "table" then
							for _ in pairs(AceLibrary.libs) do
								libs = libs + 1
							end
						end
						print(string.format(" - |cffffff7fAceLibrary instances [|r%d|cffffff7f]|r", libs))
					end
				}
			}
		})
	elseif major == "AceModuleCore-2.0" then
		AceModuleCore = instance
	end
end

local function activate(self, oldLib, oldDeactivate)
	AceAddon = self
	
	if oldLib then
		self.playerLoginFired = oldLib.playerLoginFired or DEFAULT_CHAT_FRAME and DEFAULT_CHAT_FRAME.defaultLanguage
		self.addonsToOnEnable = oldLib.addonsToOnEnable
		self.addons = oldLib.addons
		self.nextAddon = oldLib.nextAddon
		self.addonsStarted = oldLib.addonsStarted
	end
	if not self.addons then
		self.addons = {}
	end
	if not self.nextAddon then
		self.nextAddon = {}
	end
	if not self.addonsStarted then
		self.addonsStarted = {}
	end
	if oldDeactivate then
		oldDeactivate(oldLib)
	end
end

AceLibrary:Register(AceAddon, MAJOR_VERSION, MINOR_VERSION, activate, nil, external)
AceAddon = AceLibrary(MAJOR_VERSION)
```

## lib/AceConsole/AceConsole-2.0.lua
```lua
--[[
Name: AceConsole-2.0
Revision: $Rev: 15051 $
Developed by: The Ace Development Team (http://www.wowace.com/index.php/The_Ace_Development_Team)
Inspired By: Ace 1.x by Turan (turan@gryphon.com)
Website: http://www.wowace.com/
Documentation: http://www.wowace.com/index.php/AceConsole-2.0
SVN: http://svn.wowace.com/root/trunk/Ace2/AceConsole-2.0
Description: Mixin to allow for input/output capabilities. This uses the
             AceOptions data table format to determine input.
             http://wiki.wowace.com/index.php/AceOptions_data_table
Dependencies: AceLibrary, AceOO-2.0
]]

local MAJOR_VERSION = "AceConsole-2.0"
local MINOR_VERSION = "$Revision: 15051 $"

if not AceLibrary then error(MAJOR_VERSION .. " requires AceLibrary.") end
if not AceLibrary:IsNewVersion(MAJOR_VERSION, MINOR_VERSION) then return end

if not AceLibrary:HasInstance("AceOO-2.0") then error(MAJOR_VERSION .. " requires AceOO-2.0.") end

local MAP_ONOFF, USAGE, IS_CURRENTLY_SET_TO, IS_NOW_SET_TO, IS_NOT_A_VALID_OPTION_FOR, IS_NOT_A_VALID_VALUE_FOR, NO_OPTIONS_AVAILABLE, OPTION_HANDLER_NOT_FOUND, OPTION_HANDLER_NOT_VALID, OPTION_IS_DISABLED
if GetLocale() == "deDE" then
	MAP_ONOFF = { [false] = "|cffff0000Aus|r", [true] = "|cff00ff00An|r" }
	USAGE = "Benutzung"
	IS_CURRENTLY_SET_TO = "|cffffff7f%s|r steht momentan auf |cffffff7f[|r%s|cffffff7f]|r"
	IS_NOW_SET_TO = "|cffffff7f%s|r ist nun auf |cffffff7f[|r%s|cffffff7f]|r gesetzt"
	IS_NOT_A_VALID_OPTION_FOR = "[|cffffff7f%s|r] ist keine g\195\188ltige Option f\195\188r |cffffff7f%s|r"
	IS_NOT_A_VALID_VALUE_FOR = "[|cffffff7f%s|r] ist kein g\195\188ltiger Wert f\195\188r |cffffff7f%s|r"
	NO_OPTIONS_AVAILABLE = "Keine Optionen verfügbar"
	OPTION_HANDLER_NOT_FOUND = "Optionen handler |cffffff7f%q|r nicht gefunden."
	OPTION_HANDLER_NOT_VALID = "Optionen handler nicht g\195\188ltig."
	OPTION_IS_DISABLED = "Option |cffffff7f%s|r deaktiviert."
elseif GetLocale() == "frFR" then
	MAP_ONOFF = { [false] = "|cffff0000Inactif|r", [true] = "|cff00ff00Actif|r" }
	USAGE = "Utilisation"
	IS_CURRENTLY_SET_TO = "|cffffff7f%s|r est actuellement positionn\195\169 sur |cffffff7f[|r%s|cffffff7f]|r"
	IS_NOW_SET_TO = "|cffffff7f%s|r est maintenant positionn\195\169 sur |cffffff7f[|r%s|cffffff7f]|r"
	IS_NOT_A_VALID_OPTION_FOR = "[|cffffff7f%s|r] n'est pas une option valide pour |cffffff7f%s|r"
	IS_NOT_A_VALID_VALUE_FOR = "[|cffffff7f%s|r] n'est pas une valeur valide pour |cffffff7f%s|r"
	NO_OPTIONS_AVAILABLE = "Pas d'options disponibles"
	OPTION_HANDLER_NOT_FOUND = "Le gestionnaire d'option |cffffff7f%q|r n'a pas \195\169t\195\169 trouv\195\169."
	OPTION_HANDLER_NOT_VALID = "Le gestionnaire d'option n'est pas valide."
	OPTION_IS_DISABLED = "L'option |cffffff7f%s|r est d\195\169sactiv\195\169e."
elseif GetLocale() == "koKR" then
	MAP_ONOFF = { [false] = "|cffff0000끔|r", [true] = "|cff00ff00켬|r" }
	USAGE = "사용법"
	IS_CURRENTLY_SET_TO = "|cffffff7f%s|r|1은;는; 현재 상태는 |cffffff7f[|r%s|cffffff7f]|r|1으로;로; 설정되어 있습니다"
	IS_NOW_SET_TO = "|cffffff7f%s|r|1을;를; |cffffff7f[|r%s|cffffff7f]|r 상태로 변경합니다"
	IS_NOT_A_VALID_OPTION_FOR = "[|cffffff7f%s|r]|1은;는; |cffffff7f%s|r에서 사용불가능한 설정입니다"
	IS_NOT_A_VALID_VALUE_FOR = "[|cffffff7f%s|r]|1은;는; |cffffff7f%s|r에서 사용불가능한 설정값입니다"
	NO_OPTIONS_AVAILABLE = "가능한 설정이 없습니다"
	OPTION_HANDLER_NOT_FOUND = "설정 조정값인 |cffffff7f%q|r|1을;를; 찾지 못했습니다."
	OPTION_HANDLER_NOT_VALID = "설정 조정값이 올바르지 않습니다."
	OPTION_IS_DISABLED = "|cffffff7f%s|r 설정은 사용할 수 없습니다."
elseif GetLocale() == "zhCN" then
	MAP_ONOFF = { [false] = "|cffff0000\229\133\179\233\151\173|r", [true] = "|cff00ff00\229\188\128\229\144\175|r" }
	USAGE = "\231\148\168\230\179\149"
	IS_CURRENTLY_SET_TO = "|cffffff7f%s|r \229\189\147\229\137\141\232\162\171\232\174\190\231\189\174 |cffffff7f[|r%s|cffffff7f]|r"
	IS_NOW_SET_TO = "|cffffff7f%s|r \231\142\176\229\156\168\232\162\171\232\174\190\231\189\174\228\184\186 |cffffff7f[|r%s|cffffff7f]|r"
	IS_NOT_A_VALID_OPTION_FOR = "[|cffffff7f%s|r] \228\184\141\230\152\175\228\184\128\228\184\170\230\156\137\230\149\136\231\154\132\233\128\137\233\161\185 \228\184\186 |cffffff7f%s|r"
	IS_NOT_A_VALID_VALUE_FOR = "[|cffffff7f%s|r] \228\184\141\230\152\175\228\184\128\228\184\170\230\156\137\230\149\136\229\128\188 \228\184\186 |cffffff7f%s|r"
	NO_OPTIONS_AVAILABLE = "\230\178\161\230\156\137\233\128\137\233\161\185\229\143\175\231\148\168"
	OPTION_HANDLER_NOT_FOUND = "\233\128\137\233\161\185\229\164\132\231\144\134\231\168\139\229\186\143 |cffffff7f%q|r \230\178\161\230\159\165\230\137\190."
	OPTION_HANDLER_NOT_VALID = "\233\128\137\233\161\185\229\164\132\231\144\134\231\168\139\229\186\143 \230\151\160\230\149\136."
	OPTION_IS_DISABLED = "\233\128\137\233\161\185 |cffffff7f%s|r \228\184\141\229\174\140\230\149\180."
elseif GetLocale() == "zhTW" then
	MAP_ONOFF = { [false] = "|cffff0000關閉|r", [true] = "|cff00ff00開啟|r" }
	USAGE = "用法"
	IS_CURRENTLY_SET_TO = "|cffffff7f%s|r 目前的設定為 |cffffff7f[|r%s|cffffff7f]|r"
	IS_NOW_SET_TO = "|cffffff7f%s|r 現在被設定為 |cffffff7f[|r%s|cffffff7f]|r"
	IS_NOT_A_VALID_OPTION_FOR = "[|cffffff7f%s|r] 是一個不符合規定的選項，對 |cffffff7f%s|r"
	IS_NOT_A_VALID_VALUE_FOR = "[|cffffff7f%s|r] 是一個不符合規定的數值，對 |cffffff7f%s|r"
	NO_OPTIONS_AVAILABLE = "沒有可用的選項處理器。"
	OPTION_HANDLER_NOT_FOUND = "找不到 |cffffff7f%q|r 選項處理器。"
	OPTION_HANDLER_NOT_VALID = "選項處理器不符合規定。"
	OPTION_IS_DISABLED = "|cffffff7f%s|r 已被停用。"
else -- enUS
	MAP_ONOFF = { [false] = "|cffff0000Off|r", [true] = "|cff00ff00On|r" }
	USAGE = "Usage"
	IS_CURRENTLY_SET_TO = "|cffffff7f%s|r is currently set to |cffffff7f[|r%s|cffffff7f]|r"
	IS_NOW_SET_TO = "|cffffff7f%s|r is now set to |cffffff7f[|r%s|cffffff7f]|r"
	IS_NOT_A_VALID_OPTION_FOR = "[|cffffff7f%s|r] is not a valid option for |cffffff7f%s|r"
	IS_NOT_A_VALID_VALUE_FOR = "[|cffffff7f%s|r] is not a valid value for |cffffff7f%s|r"
	NO_OPTIONS_AVAILABLE = "No options available"
	OPTION_HANDLER_NOT_FOUND = "Option handler |cffffff7f%q|r not found."
	OPTION_HANDLER_NOT_VALID = "Option handler not valid."
	OPTION_IS_DISABLED = "Option |cffffff7f%s|r is disabled."
end

local NONE = NONE or "None"

local AceOO = AceLibrary("AceOO-2.0")
local AceEvent

local AceConsole = AceOO.Mixin { "Print", "PrintComma", "CustomPrint", "RegisterChatCommand" }
local Dewdrop

local _G = getfenv(0)

local table_setn
do
	local version = GetBuildInfo()
	if string.find(version, "^2%.") then
		-- 2.0.0
		table_setn = function() end
	else
		table_setn = table.setn
	end
end

local function print(text, name, r, g, b, frame, delay)
	if not text or string.len(text) == 0 then
		text = " "
	end
	if not name or name == AceConsole then
		(frame or DEFAULT_CHAT_FRAME):AddMessage(text, r, g, b, nil, delay or 5)
	else
		(frame or DEFAULT_CHAT_FRAME):AddMessage("|cffffff78" .. tostring(name) .. ":|r " .. text, r, g, b, nil, delay or 5)
	end
end

local tmp
function AceConsole:CustomPrint(r, g, b, frame, delay, connector, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	a1 = tostring(a1)
	if string.find(a1, "%%") then
		print(string.format(a1, tostring(a2), tostring(a3), tostring(a4), tostring(a5), tostring(a6), tostring(a7), tostring(a8), tostring(a9), tostring(a10), tostring(a11), tostring(a12), tostring(a13), tostring(a14), tostring(a15), tostring(a16), tostring(a17), tostring(a18), tostring(a19), tostring(a20)), self, r, g, b, frame or self.printFrame, delay)
	else
		if not tmp then
			tmp = {}
		end
		tmp[1] = a1
		tmp[2] = a2
		tmp[3] = a3
		tmp[4] = a4
		tmp[5] = a5
		tmp[6] = a6
		tmp[7] = a7
		tmp[8] = a8
		tmp[9] = a9
		tmp[10] = a10
		tmp[11] = a11
		tmp[12] = a12
		tmp[13] = a13
		tmp[14] = a14
		tmp[15] = a15
		tmp[16] = a16
		tmp[17] = a17
		tmp[18] = a18
		tmp[19] = a19
		tmp[20] = a20
		local n = 20
		while tmp[n] == nil do
			n = n - 1
		end
		table_setn(tmp, n)
		for k = 1, n do
			tmp[k] = tostring(tmp[k])
		end
		print(table.concat(tmp, connector or " "), self, r, g, b, frame or self.printFrame, delay)
		for k,v in pairs(tmp) do
			tmp[k] = nil
		end
		table_setn(tmp, 0)
	end
end

function AceConsole:Print(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	return AceConsole.CustomPrint(self, nil, nil, nil, nil, nil, " ", a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
end

function AceConsole:PrintComma(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	return AceConsole.CustomPrint(self, nil, nil, nil, nil, nil, ", ", a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
end

local work
local argwork

local string_gfind = string.gmatch or string.gfind

local function findTableLevel(self, options, chat, text, index, passTable)
	if not index then
		index = 1
		if work then
			for k,v in pairs(work) do
				work[k] = nil
			end
			table_setn(work, 0)
			for k,v in pairs(argwork) do
				argwork[k] = nil
			end
			table_setn(argwork, 0)
		else
			work = {}
			argwork = {}
		end
		local len = string.len(text)
		local count
		repeat
			text, count = string.gsub(text, "(|cff%x%x%x%x%x%x|Hitem:%d-:%d-:%d-:%d-|h%[[^%]]-) (.-%]|h|r)", "%1\001%2")
		until count == 0
		text = string.gsub(text, "(%]|h|r)(|cff%x%x%x%x%x%x|Hitem:%d-:%d-:%d-:%d-|h%[)", "%1 %2")
		for token in string_gfind(text, "([^%s]+)") do
			local token = token
			local num = tonumber(token)
			if num then
				token = num
			else
				token = string.gsub(token, "\001", " ")
			end
			table.insert(work, token)
		end
	end

	local path = chat
	for i = 1, index - 1 do
		path = path .. " " .. tostring(work[i])
	end

	if type(options.args) == "table" then
		local disabled, hidden = options.disabled, options.cmdHidden or options.hidden
		if hidden then
			if type(hidden) == "function" then
				hidden = hidden()
			elseif type(hidden) == "string" then
				local handler = options.handler or self
				if type(handler[hidden]) ~= "function" then
					AceConsole:error(OPTION_HANDLER_NOT_FOUND, hidden)
				end
				hidden = handler[hidden](handler)
			end
		end
		if hidden then
			disabled = true
		elseif disabled then
			if type(disabled) == "function" then
				disabled = disabled()
			elseif type(disabled) == "string" then
				local handler = options.handler or self
				if type(handler[disabled]) ~= "function" then
					AceConsole:error(OPTION_HANDLER_NOT_FOUND, disabled)
				end
				disabled = handler[disabled](handler)
			end
		end
		if not disabled then
			local next = work[index] and string.lower(work[index])
			if next then
				for k,v in pairs(options.args) do
					local good = false
					if string.lower(k) == next then
						good = true
					elseif type(v.aliases) == "table" then
						for _,alias in ipairs(v.aliases) do
							if string.lower(alias) == next then
								good = true
								break
							end
						end
					elseif type(v.aliases) == "string" and string.lower(v.aliases) == next then
						good = true
					end
					if good then
						return findTableLevel(options.handler or self, v, chat, text, index + 1, options.pass and options or nil)
					end
				end
			end
		end
	end
	for i = index, table.getn(work) do
		table.insert(argwork, work[i])
	end
	return options, path, argwork, options.handler or self, passTable, passTable and work[index - 1]
end

local function validateOptionsMethods(self, options, position)
	if type(options) ~= "table" then
		return "Options must be a table.", position
	end
	self = options.handler or self
	if options.type == "execute" then
		if options.func and type(options.func) ~= "string" and type(options.func) ~= "function" then
			return "func must be a string or function", position
		end
		if options.func and type(options.func) == "string" and type(self[options.func]) ~= "function" then
			return string.format("%q is not a proper function", options.func), position
		end
	else
		if options.get then
			if type(options.get) ~= "string" and type(options.get) ~= "function" then
				return "get must be a string or function", position
			end
			if type(options.get) == "string" and type(self[options.get]) ~= "function" then
				return string.format("%q is not a proper function", options.get), position
			end
		end
		if options.set then
			if type(options.set) ~= "string" and type(options.set) ~= "function" then
				return "set must be a string or function", position
			end
			if type(options.set) == "string" and type(self[options.set]) ~= "function" then
				return string.format("%q is not a proper function", options.set), position
			end
		end
		if options.validate and type(options.validate) ~= "table" then
			if type(options.validate) ~= "string" and type(options.validate) ~= "function" then
				return "validate must be a string or function", position
			end
			if type(options.validate) == "string" and type(self[options.validate]) ~= "function" then
				return string.format("%q is not a proper function", options.validate), position
			end
		end
	end
	if options.disabled and type(options.disabled) == "string" and type(self[options.disabled]) ~= "function" then
		return string.format("%q is not a proper function", options.disabled), position
	end
	if options.cmdHidden and type(options.cmdHidden) == "string" and type(self[options.cmdHidden]) ~= "function" then
		return string.format("%q is not a proper function", options.cmdHidden), position
	end
	if options.guiHidden and type(options.guiHidden) == "string" and type(self[options.guiHidden]) ~= "function" then
		return string.format("%q is not a proper function", options.guiHidden), position
	end
	if options.hidden and type(options.hidden) == "string" and type(self[options.hidden]) ~= "function" then
		return string.format("%q is not a proper function", options.hidden), position
	end
	if options.type == "group" and type(options.args) == "table" then
		for k,v in pairs(options.args) do
			if type(v) == "table" then
				local newposition
				if position then
					newposition = position .. ".args." .. k
				else
					newposition = "args." .. k
				end
				local err, pos = validateOptionsMethods(self, v, newposition)
				if err then
					return err, pos
				end
			end
		end
	end
end

local function validateOptions(options, position, baseOptions, fromPass)
	if not baseOptions then
		baseOptions = options
	end
	if type(options) ~= "table" then
		return "Options must be a table.", position
	end
	local kind = options.type
	if type(kind) ~= "string" then
		return '"type" must be a string.', position
	elseif kind ~= "group" and kind ~= "range" and kind ~= "text" and kind ~= "execute" and kind ~= "toggle" and kind ~= "color" and kind ~= "header" then
		return '"type" must either be "range", "text", "group", "toggle", "execute", "color", or "header".', position
	end
	if options.aliases then
		if type(options.aliases) ~= "table" and type(options.aliases) ~= "string" then
			return '"alias" must be a table or string', position
		end
	end
	if not fromPass then
		if kind == "execute" then
			if type(options.func) ~= "string" and type(options.func) ~= "function" then
				return '"func" must be a string or function', position
			end
		elseif kind == "range" or kind == "text" or kind == "toggle" then
			if type(options.set) ~= "string" and type(options.set) ~= "function" then
				return '"set" must be a string or function', position
			end
			if kind == "text" and options.get == false then
			elseif type(options.get) ~= "string" and type(options.get) ~= "function" then
				return '"get" must be a string or function', position
			end
		elseif kind == "group" and options.pass then
			if options.pass ~= true then
				return '"pass" must be either nil, true, or false', position
			end
			if not options.func then
				if type(options.set) ~= "string" and type(options.set) ~= "function" then
					return '"set" must be a string or function', position
				end
				if type(options.get) ~= "string" and type(options.get) ~= "function" then
					return '"get" must be a string or function', position
				end
			elseif type(options.func) ~= "string" and type(options.func) ~= "function" then
				return '"func" must be a string or function', position
			end
		end
	else
		if kind == "group" then
			return 'cannot have "type" = "group" as a subgroup of a passing group', position
		end
	end
	if options ~= baseOptions then
		if kind == "header" then
		elseif type(options.desc) ~= "string" then
			return '"desc" must be a string', position
		elseif string.len(options.desc) == 0 then
			return '"desc" cannot be a 0-length string', position
		end
	end

	if options ~= baseOptions or kind == "range" or kind == "text" or kind == "toggle" or kind == "color" then
		if options.type == "header" and not options.cmdName and not options.name then
		elseif options.cmdName then
			if type(options.cmdName) ~= "string" then
				return '"cmdName" must be a string or nil', position
			elseif string.len(options.cmdName) == 0 then
				return '"cmdName" cannot be a 0-length string', position
			end
			if type(options.guiName) ~= "string" then
				if not options.guiNameIsMap then
					return '"guiName" must be a string or nil', position
				end
			elseif string.len(options.guiName) == 0 then
				return '"guiName" cannot be a 0-length string', position
			end
		else
			if type(options.name) ~= "string" then
				return '"name" must be a string', position
			elseif string.len(options.name) == 0 then
				return '"name" cannot be a 0-length string', position
			end
		end
	end
	if options.guiNameIsMap then
		if type(options.guiNameIsMap) ~= "boolean" then
			return '"guiNameIsMap" must be a boolean or nil', position
		elseif options.type ~= "toggle" then
			return 'if "guiNameIsMap" is true, then "type" must be set to \'toggle\'', position
		elseif type(options.map) ~= "table" then
			return '"map" must be a table', position
		end
	end
	if options.message and type(options.message) ~= "string" then
		return '"message" must be a string or nil', position
	end
	if options.error and type(options.error) ~= "string" then
		return '"error" must be a string or nil', position
	end
	if options.current and type(options.current) ~= "string" then
		return '"current" must be a string or nil', position
	end
	if options.order then
		if type(options.order) ~= "number" or (-1 < options.order and options.order < 0.999) then
			return '"order" must be a non-zero number or nil', position
		end
	end
	if options.disabled then
		if type(options.disabled) ~= "function" and type(options.disabled) ~= "string" and options.disabled ~= true then
			return '"disabled" must be a function, string, or boolean', position
		end
	end
	if options.cmdHidden then
		if type(options.cmdHidden) ~= "function" and type(options.cmdHidden) ~= "string" and options.cmdHidden ~= true then
			return '"cmdHidden" must be a function, string, or boolean', position
		end
	end
	if options.guiHidden then
		if type(options.guiHidden) ~= "function" and type(options.guiHidden) ~= "string" and options.guiHidden ~= true then
			return '"guiHidden" must be a function, string, or boolean', position
		end
	end
	if options.hidden then
		if type(options.hidden) ~= "function" and type(options.hidden) ~= "string" and options.hidden ~= true then
			return '"hidden" must be a function, string, or boolean', position
		end
	end
	if kind == "text" then
		if type(options.validate) == "table" then
			local t = options.validate
			local iTable = nil
			for k,v in pairs(t) do
				if type(k) == "number" then
					if iTable == nil then
						iTable = true
					elseif not iTable then
						return '"validate" must either have all keys be indexed numbers or strings', position
					elseif k < 1 or k > table.getn(t) then
						return '"validate" numeric keys must be indexed properly. >= 1 and <= table.getn', position
					end
				else
					if iTable == nil then
						iTable = false
					elseif iTable then
						return '"validate" must either have all keys be indexed numbers or strings', position
					end
				end
				if type(v) ~= "string" then
					return '"validate" values must all be strings', position
				end
			end
		else
			if type(options.usage) ~= "string" then
				return '"usage" must be a string', position
			elseif options.validate and type(options.validate) ~= "string" and type(options.validate) ~= "function" then
				return '"validate" must be a string, function, or table', position
			end
		end
	elseif kind == "range" then
		if options.min or options.max then
			if type(options.min) ~= "number" then
				return '"min" must be a number', position
			elseif type(options.max) ~= "number" then
				return '"max" must be a number', position
			elseif options.min >= options.max then
				return '"min" must be less than "max"', position
			end
		end
		if options.step then
			if type(options.step) ~= "number" then
				return '"step" must be a number', position
			elseif options.step < 0 then
				return '"step" must be nonnegative', position
			end
		end
		if options.isPercent and options.isPercent ~= true then
			return '"isPercent" must either be nil, true, or false', position
		end
	elseif kind == "toggle" then
		if options.map then
			if type(options.map) ~= "table" then
				return '"map" must be a table', position
			elseif type(options.map[true]) ~= "string" then
				return '"map[true]" must be a string', position
			elseif type(options.map[false]) ~= "string" then
				return '"map[false]" must be a string', position
			end
		end
	elseif kind == "color" then
		if options.hasAlpha and options.hasAlpha ~= true then
			return '"hasAlpha" must be nil, true, or false', position
		end
	elseif kind == "group" then
		if options.pass and options.pass ~= true then
			return '"pass" must be nil, true, or false', position
		end
		if type(options.args) ~= "table" then
			return '"args" must be a table', position
		end
		for k,v in pairs(options.args) do
			if type(k) ~= "string" then
				return '"args" keys must be strings', position
			elseif string.find(k, "%s") then
				return string.format('"args" keys must not include spaces. %q is not appropriate.', k), position
			elseif string.len(k) == 0 then
				return '"args" keys must not be 0-length strings.', position
			end
			if type(v) ~= "table" then
				return '"args" values must be tables', position and position .. "." .. k or k
			end
			local newposition
			if position then
				newposition = position .. ".args." .. k
			else
				newposition = "args." .. k
			end
			local err, pos = validateOptions(v, newposition, baseOptions, options.pass)
			if err then
				return err, pos
			end
		end
	end
end

local colorTable
local colorFunc
local colorCancelFunc

local order

local mysort_args
local mysort

local function printUsage(self, handler, realOptions, options, path, args, quiet, filter)
	if filter then
		filter = "^" .. string.gsub(filter, "([%(%)%.%*%+%-%[%]%?%^%$%%])", "%%%1")
	end
	local hidden, disabled = options.cmdHidden or options.hidden, options.disabled
	if hidden then
		if type(hidden) == "function" then
			hidden = hidden()
		elseif type(hidden) == "string" then
			if type(handler[handler]) ~= "function" then
				AceConsole:error(OPTION_HANDLER_NOT_FOUND, handler)
			end
			hidden = handler[hidden](handler)
		end
	end
	if hidden then
		disabled = true
	elseif disabled then
		if type(disabled) == "function" then
			disabled = disabled()
		elseif type(disabled) == "string" then
			if type(handler[disabled]) ~= "function" then
				AceConsole:error(OPTION_HANDLER_NOT_FOUND, disabled)
			end
			disabled = handler[disabled](handler)
		end
	end
	local kind = string.lower(options.type or "group")
	if disabled then
		print(string.format(OPTION_IS_DISABLED, path), realOptions.cmdName or realOptions.name or self)
	elseif kind == "text" then
		local var
		if passTable then
			if not passTable.get then
			elseif type(passTable.get) == "function" then
				var = passTable.get(passValue)
			else
				local handler = passTable.handler or handler
				if type(handler[passTable.get]) ~= "function" then
					AceConsole:error(OPTION_HANDLER_NOT_FOUND, passTable.get)
				end
				var = handler[passTable.get](handler, passValue)
			end
		else
			if not options.get then
			elseif type(options.get) == "function" then
				var = options.get()
			else
				local handler = options.handler or handler
				if type(handler[options.get]) ~= "function" then
					AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.get)
				end
				var = handler[options.get](handler)
			end
		end

		local usage
		if type(options.validate) == "table" then
			if filter then
				if not order then
					order = {}
				end
				for k,v in pairs(options.validate) do
					if string.find(v, filter) then
						table.insert(order, v)
					end
				end
				usage = "{" .. table.concat(order, " || ") .. "}"
				for k in pairs(order) do
					order[k] = nil
				end
				table_setn(order, 0)
			else
				if not order then
					order = {}
				end
				for k,v in pairs(options.validate) do
					table.insert(order, v)
				end
				usage = "{" .. table.concat(order, " || ") .. "}"
				for k in pairs(order) do
					order[k] = nil
				end
				table_setn(order, 0)
			end
			var = options.validate[var] or var
		else
			usage = options.usage or "<value>"
		end
		if not quiet then
			print(string.format("|cffffff7f%s:|r %s %s", USAGE, path, usage), realOptions.cmdName or realOptions.name or self)
		end
		if (passTable and passTable.get) or options.get then
			print(string.format(options.current or IS_CURRENTLY_SET_TO, tostring(options.cmdName or options.name), tostring(var or NONE)))
		end
	elseif kind == "range" then
		local var
		if passTable then
			if type(passTable.get) == "function" then
				var = passTable.get(passValue)
			else
				local handler = passTable.handler or handler
				if type(handler[passTable.get]) ~= "function" then
					AceConsole:error(OPTION_HANDLER_NOT_FOUND, passTable.get)
				end
				var = handler[passTable.get](handler, passValue)
			end
		else
			if type(options.get) == "function" then
				var = options.get()
			else
				local handler = options.handler or handler
				if type(handler[options.get]) ~= "function" then
					AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.get)
				end
				var = handler[options.get](handler)
			end
		end

		local usage
		local min = options.min or 0
		local max = options.max or 1
		if options.isPercent then
			min, max = min * 100, max * 100
			var = tostring(var * 100) .. "%"
		end
		local bit = "-"
		if min < 0 or max < 0 then
			bit = " - "
		end
		usage = string.format("(%s%s%s)", min, bit, max)
		if not quiet then
			print(string.format("|cffffff7f%s:|r %s %s", USAGE, path, usage), realOptions.cmdName or realOptions.name or self)
		end
		print(string.format(options.current or IS_CURRENTLY_SET_TO, tostring(options.cmdName or options.name), tostring(var or NONE)))
	elseif kind == "group" then
		local usage
		if next(options.args) then
			if not order then
				order = {}
			end
			for k,v in pairs(options.args) do
				if v.type ~= "header" then
					local hidden = v.cmdHidden or v.hidden
					if hidden then
						if type(hidden) == "function" then
							hidden = hidden()
						elseif type(hidden) == "string" then
							local handler = v.handler or handler
							if type(handler[hidden]) ~= "function" then
								AceConsole:error(OPTION_HANDLER_NOT_FOUND, hidden)
							end
							hidden = handler[hidden](handler)
						end
					end
					if not hidden then
						if filter then
							if string.find(k, filter) then
								table.insert(order, k)
							elseif type(v.aliases) == "table" then
								for _,bit in ipairs(v.aliases) do
									if string.find(v.aliases, filter) then
										table.insert(order, k)
										break
									end
								end
							elseif type(v.aliases) == "string" then
								if string.find(v.aliases, filter) then
									table.insert(order, k)
								end
							end
						else
							table.insert(order, k)
						end
					end
				end
			end
			if not mysort then
				mysort = function(a, b)
					local alpha, bravo = mysort_args[a], mysort_args[b]
					local alpha_order = alpha and alpha.order or 100
					local bravo_order = bravo and bravo.order or 100
					if alpha_order == bravo_order then
						return tostring(a) < tostring(b)
					else
						if alpha_order < 0 then
							if bravo_order > 0 then
								return false
							end
						else
							if bravo_order < 0 then
								return true
							end
						end
						if alpha_order > 0 and bravo_order > 0 then
							return tostring(a) < tostring(b)
						end
						return alpha_order < bravo_order
					end
				end
			end
			mysort_args = options.args
			table.sort(order, mysort)
			mysort_args = nil
			if not quiet then
				if options == realOptions then
					if options.desc then
						print(tostring(options.desc), realOptions.cmdName or realOptions.name or self)
						print(string.format("|cffffff7f%s:|r %s %s", USAGE, path, "{" .. table.concat(order, " || ") .. "}"))
					elseif self.description or self.notes then
						print(tostring(self.description or self.notes), realOptions.cmdName or realOptions.name or self)
						print(string.format("|cffffff7f%s:|r %s %s", USAGE, path, "{" .. table.concat(order, " || ") .. "}"))
					else
						print(string.format("|cffffff7f%s:|r %s %s", USAGE, path, "{" .. table.concat(order, " || ") .. "}"), realOptions.cmdName or realOptions.name or self)
					end
				else
					if options.desc then
						print(string.format("|cffffff7f%s:|r %s %s", USAGE, path, "{" .. table.concat(order, " || ") .. "}"), realOptions.cmdName or realOptions.name or self)
						print(tostring(options.desc))
					else
						print(string.format("|cffffff7f%s:|r %s %s", USAGE, path, "{" .. table.concat(order, " || ") .. "}"), realOptions.cmdName or realOptions.name or self)
					end
				end
			end
			for _,k in ipairs(order) do
				local v = options.args[k]
				if v then
					local disabled = v.disabled
					if disabled then
						if type(disabled) == "function" then
							disabled = disabled()
						elseif type(disabled) == "string" then
							local handler = v.handler or handler
							if type(handler[disabled]) ~= "function" then
								AceConsole:error(OPTION_HANDLER_NOT_FOUND, disabled)
							end
							disabled = handler[disabled](handler)
						end
					end
					if type(v.aliases) == "table" then
						k = k .. " || " .. table.concat(v.aliases, " || ")
					elseif type(v.aliases) == "string" then
						k = k .. " || " .. v.aliases
					end
					if v.get then
						local a1,a2,a3,a4
						if type(v.get) == "function" then
							if options.pass then
								a1,a2,a3,a4 = v.get(k)
							else
								a1,a2,a3,a4 = v.get()
							end
						else
							local handler = v.handler or handler
							if type(handler[v.get]) ~= "function" then
								AceConsole:error(OPTION_HANDLER_NOT_FOUND, v.get)
							end
							if options.pass then
								a1,a2,a3,a4 = handler[v.get](handler, k)
							else
								a1,a2,a3,a4 = handler[v.get](handler)
							end
						end
						if v.type == "color" then
							if v.hasAlpha then
								if not a1 or not a2 or not a3 or not a4 then
									s = NONE
								else
									s = string.format("|c%02x%02x%02x%02x%02x%02x%02x%02x|r", a4*255, a1*255, a2*255, a3*255, a4*255, a1*255, a2*255, a3*255)
								end
							else
								if not a1 or not a2 or not a3 then
									s = NONE
								else
									s = string.format("|cff%02x%02x%02x%02x%02x%02x|r", a1*255, a2*255, a3*255, a1*255, a2*255, a3*255)
								end
							end
						elseif v.type == "toggle" then
							if v.map then
								s = tostring(v.map[a1 and true or false] or NONE)
							else
								s = tostring(MAP_ONOFF[a1 and true or false] or NONE)
							end
						elseif v.type == "range" then
							if v.isPercent then
								s = tostring(a1 * 100) .. "%"
							else
								s = tostring(a1)
							end
						elseif v.type == "text" and type(v.validate) == "table" then
							s = tostring(v.validate[a1] or a1)
						else
							s = tostring(a1 or NONE)
						end
						if disabled then
							local s = string.gsub(s, "|cff%x%x%x%x%x%x(.-)|r", "%1")
							local desc = string.gsub(v.desc or NONE, "|cff%x%x%x%x%x%x(.-)|r", "%1")
							print(string.format("|cffcfcfcf - %s: [%s] %s|r", k, s, desc))
						else
							print(string.format(" - |cffffff7f%s: [|r%s|cffffff7f]|r %s", k, s, v.desc or NONE))
						end
					else
						if disabled then
							local desc = string.gsub(v.desc or NONE, "|cff%x%x%x%x%x%x(.-)|r", "%1")
							print(string.format("|cffcfcfcf - %s: %s", k, desc))
						else
							print(string.format(" - |cffffff7f%s:|r %s", k, v.desc or NONE))
						end
					end
				end
			end
			for k in pairs(order) do
				order[k] = nil
			end
			table_setn(order, 0)
		else
			if options.desc then
				desc = options.desc
				print(string.format("|cffffff7f%s:|r %s", USAGE, path), realOptions.cmdName or realOptions.name or self)
				print(tostring(options.desc))
			elseif options == realOptions and (self.description or self.notes) then
				print(tostring(self.description or self.notes), realOptions.cmdName or realOptions.name or self)
				print(string.format("|cffffff7f%s:|r %s", USAGE, path))
			else
				print(string.format("|cffffff7f%s:|r %s", USAGE, path), realOptions.cmdName or realOptions.name or self)
			end
			print(self, NO_OPTIONS_AVAILABLE)
		end
	end
end

local function handlerFunc(self, chat, msg, options)
	if not msg then
		msg = ""
	else
		msg = string.gsub(msg, "^%s*(.-)%s*$", "%1")
		msg = string.gsub(msg, "%s+", " ")
	end

	local realOptions = options
	local options, path, args, handler, passTable, passValue = findTableLevel(self, options, chat, msg)

	local hidden, disabled = options.cmdHidden or options.hidden, options.disabled
	if hidden then
		if type(hidden) == "function" then
			hidden = hidden()
		elseif type(hidden) == "string" then
			if type(handler[hidden]) ~= "function" then
				AceConsole:error(OPTION_HANDLER_NOT_FOUND, hidden)
			end
			hidden = handler[hidden](handler)
		end
	end
	if hidden then
		disabled = true
	elseif disabled then
		if type(disabled) == "function" then
			disabled = disabled()
		elseif type(disabled) == "string" then
			if type(handler[disabled]) ~= "function" then
				AceConsole:error(OPTION_HANDLER_NOT_FOUND, disabled)
			end
			disabled = handler[disabled](handler)
		end
	end
	local _G_this = this
	local kind = string.lower(options.type or "group")
	if disabled then
		print(string.format(OPTION_IS_DISABLED, path), realOptions.cmdName or realOptions.name or self)
	elseif kind == "text" then
		if table.getn(args) > 0 then
			if (type(options.validate) == "table" and table.getn(args) > 1) or (type(options.validate) ~= "table" and not options.input) then
				local arg = table.concat(args, " ")
				for k,v in pairs(args) do
					args[k] = nil
				end
				table_setn(args, 0)
				table.insert(args, arg)
			end
			if options.validate then
				local good
				if type(options.validate) == "function" then
					good = options.validate(unpack(args))
				elseif type(options.validate) == "table" then
					local arg = args[1]
					arg = string.lower(tostring(arg))
					for k,v in pairs(options.validate) do
						if string.lower(v) == arg then
							args[1] = type(k) == "string" and k or v
							good = true
							break
						end
					end
					if not good and type((next(options.validate))) == "string" then
						for k,v in pairs(options.validate) do
							if type(k) == "string" and string.lower(k) == arg then
								args[1] = k
								good = true
								break
							end
						end
					end
				else
					if type(handler[options.validate]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.validate)
					end
					good = handler[options.validate](handler, unpack(args))
				end
				if not good then
					local usage
					if type(options.validate) == "table" then
						if not order then
							order = {}
						end
						for k,v in pairs(options.validate) do
							table.insert(order, v)
						end
						usage = "{" .. table.concat(order, " || ") .. "}"
						for k in pairs(order) do
							order[k] = nil
						end
						table_setn(order, 0)
					else
						usage = options.usage or "<value>"
					end
					print(string.format(options.error or IS_NOT_A_VALID_OPTION_FOR, tostring(table.concat(args, " ")), path), realOptions.cmdName or realOptions.name or self)
					print(string.format("|cffffff7f%s:|r %s %s", USAGE, path, usage))
					return
				end
			end

			local var
			if passTable then
				if not passTable.get then
				elseif type(passTable.get) == "function" then
					var = passTable.get(passValue)
				else
					if type(handler[passTable.get]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, passTable.get)
					end
					var = handler[passTable.get](handler, passValue)
				end
			else
				if not options.get then
				elseif type(options.get) == "function" then
					var = options.get()
				else
					if type(handler[options.get]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.get)
					end
					var = handler[options.get](handler)
				end
			end

			if var ~= args[1] then
				if passTable then
					if type(passTable.set) == "function" then
						passTable.set(passValue, unpack(args))
					else
						if type(handler[passTable.set]) ~= "function" then
							AceConsole:error(OPTION_HANDLER_NOT_FOUND, passTable.set)
						end
						handler[passTable.set](handler, passTable.set, unpack(args))
					end
				else
					if type(options.set) == "function" then
						options.set(unpack(args))
					else
						if type(handler[options.set]) ~= "function" then
							AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.set)
						end
						handler[options.set](handler, unpack(args))
					end
				end
			end
		end

		if table.getn(args) > 0 then
			local var
			if passTable then
				if not passTable.get then
				elseif type(passTable.get) == "function" then
					var = passTable.get(passValue)
				else
					if type(handler[passTable.get]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, passTable.get)
					end
					var = handler[passTable.get](handler, passValue)
				end
			else
				if not options.get then
				elseif type(options.get) == "function" then
					var = options.get()
				else
					if type(handler[options.get]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.get)
					end
					var = handler[options.get](handler)
				end
			end
			if type(options.validate) == "table" then
				var = options.validate[var] or var
			end
			if (passTable and passTable.get) or options.get then
				print(string.format(options.message or IS_NOW_SET_TO, tostring(options.cmdName or options.name), tostring(var or NONE)), realOptions.cmdName or realOptions.name or self)
			end
			if var == args[1] then
				return
			end
		else
			printUsage(self, handler, realOptions, options, path, args)
			return
		end
	elseif kind == "execute" then
		if passTable then
			if type(passFunc) == "function" then
				set(passValue)
			else
				if type(handler[passFunc]) ~= "function" then
					AceConsole:error(OPTION_HANDLER_NOT_FOUND, passFunc)
				end
				handler[passFunc](handler, passValue)
			end
		else
			local ret, msg
			if type(options.func) == "function" then
				options.func()
			else
				local handler = options.handler or self
				if type(handler[options.func]) ~= "function" then
					AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.func)
				end
				handler[options.func](handler)
			end
		end
	elseif kind == "toggle" then
		local var
		if passTable then
			if type(passTable.get) == "function" then
				var = passTable.get(passValue)
			else
				if type(handler[passTable.get]) ~= "function" then
					AceConsole:error(OPTION_HANDLER_NOT_FOUND, passTable.get)
				end
				var = handler[passTable.get](handler, passValue)
			end
			if type(passTable.set) == "function" then
				passTable.set(passValue, not var)
			else
				if type(handler[passTable.set]) ~= "function" then
					AceConsole:error(OPTION_HANDLER_NOT_FOUND, passTable.set)
				end
				handler[passTable.set](handler, passValue, not var)
			end
			if type(passTable.get) == "function" then
				var = passTable.get(passValue)
			else
				var = handler[passTable.get](handler, passValue)
			end
		else
			local handler = options.handler or self
			if type(options.get) == "function" then
				var = options.get()
			else
				if type(handler[options.get]) ~= "function" then
					AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.get)
				end
				var = handler[options.get](handler)
			end
			if type(options.set) == "function" then
				options.set(not var)
			else
				if type(handler[options.set]) ~= "function" then
					AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.set)
				end
				handler[options.set](handler, not var)
			end
			if type(options.get) == "function" then
				var = options.get()
			else
				var = handler[options.get](handler)
			end
		end

		print(string.format(options.message or IS_NOW_SET_TO, tostring(options.cmdName or options.name), (options.map or MAP_ONOFF)[var and true or false] or NONE), realOptions.cmdName or realOptions.name or self)
	elseif kind == "range" then
		local arg
		if table.getn(args) <= 1 then
			arg = args[1]
		else
			arg = table.concat(args, " ")
		end

		if arg then
			local min = options.min or 0
			local max = options.max or 1
			local good = false
			if type(arg) == "number" then
				if options.isPercent then
					arg = arg / 100
				end

				if arg >= min and arg <= max then
					good = true
				end

				if good and type(options.step) == "number" and options.step > 0 then
					local step = options.step
					arg = math.floor((arg - min) / step + 0.5) * step + min
					if arg > max then
						arg = max
					elseif arg < min then
						arg = min
					end
				end
			end
			if not good then
				local usage
				local min = options.min or 0
				local max = options.max or 1
				if options.isPercent then
					min, max = min * 100, max * 100
				end
				local bit = "-"
				if min < 0 or max < 0 then
					bit = " - "
				end
				usage = string.format("(%s%s%s)", min, bit, max)
				print(string.format(options.error or IS_NOT_A_VALID_VALUE_FOR, tostring(arg), path), realOptions.cmdName or realOptions.name or self)
				print(string.format("|cffffff7f%s:|r %s %s", USAGE, path, usage))
				return
			end

			local var
			if passTable then
				if type(passTable.get) == "function" then
					var = passTable.get(passValue)
				else
					if type(handler[passTable.get]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, passTable.get)
					end
					var = handler[passTable.get](handler, passValue)
				end
			else
				if type(options.get) == "function" then
					var = options.get()
				else
					local handler = options.handler or self
					if type(handler[options.get]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.get)
					end
					var = handler[options.get](handler)
				end
			end

			if var ~= arg then
				if passTable then
					if type(passTable.set) == "function" then
						passTable.set(passValue, arg)
					else
						if type(handler[passTable.set]) ~= "function" then
							AceConsole:error(OPTION_HANDLER_NOT_FOUND, passTable.set)
						end
						handler[passTable.set](handler, passValue, arg)
					end
				else
					if type(options.set) == "function" then
						options.set(arg)
					else
						local handler = options.handler or self
						if type(handler[options.set]) ~= "function" then
							AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.set)
						end
						handler[options.set](handler, arg)
					end
				end
			end
		end

		if arg then
			local var
			if passTable then
				if type(passTable.get) == "function" then
					var = passTable.get(passValue)
				else
					if type(handler[passTable.get]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, passTable.get)
					end
					var = handler[passTable.get](handler, passValue)
				end
			else
				if type(options.get) == "function" then
					var = options.get()
				else
					local handler = options.handler or self
					if type(handler[options.get]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.get)
					end
					var = handler[options.get](handler)
				end
			end

			if var and options.isPercent then
				var = tostring(var * 100) .. "%"
			end
			print(string.format(options.message or IS_NOW_SET_TO, tostring(options.cmdName or options.name), tostring(var or NONE)), realOptions.cmdName or realOptions.name or self)
			if var == arg then
				return
			end
		else
			printUsage(self, handler, realOptions, options, path, args)
			return
		end
	elseif kind == "color" then
		if table.getn(args) > 0 then
			local r,g,b,a
			if table.getn(args) == 1 then
				local arg = tostring(args[1])
				if options.hasAlpha then
					if string.len(arg) == 8 and string.find(arg, "^%x*$")  then
						r,g,b,a = tonumber(string.sub(arg, 1, 2), 16) / 255, tonumber(string.sub(arg, 3, 4), 16) / 255, tonumber(string.sub(arg, 5, 6), 16) / 255, tonumber(string.sub(arg, 7, 8), 16) / 255
					end
				else
					if string.len(arg) == 6 and string.find(arg, "^%x*$") then
						r,g,b = tonumber(string.sub(arg, 1, 2), 16) / 255, tonumber(string.sub(arg, 3, 4), 16) / 255, tonumber(string.sub(arg, 5, 6), 16) / 255
					end
				end
			elseif table.getn(args) == 4 and options.hasAlpha then
				local a1,a2,a3,a4 = args[1], args[2], args[3], args[4]
				if type(a1) == "number" and type(a2) == "number" and type(a3) == "number" and type(a4) == "number" and a1 <= 1 and a2 <= 1 and a3 <= 1 and a4 <= 1 then
					r,g,b,a = a1,a2,a3,a4
				elseif (type(a1) == "number" or string.len(a1) == 2) and string.find(a1, "^%x*$") and (type(a2) == "number" or string.len(a2) == 2) and string.find(a2, "^%x*$") and (type(a3) == "number" or string.len(a3) == 2) and string.find(a3, "^%x*$") and (type(a4) == "number" or string.len(a4) == 2) and string.find(a4, "^%x*$") then
					r,g,b,a = tonumber(a1, 16) / 255, tonumber(a2, 16) / 255, tonumber(a3, 16) / 255, tonumber(a4, 16) / 255
				end
			elseif table.getn(args) == 3 and not options.hasAlpha then
				local a1,a2,a3 = args[1], args[2], args[3]
				if type(a1) == "number" and type(a2) == "number" and type(a3) == "number" and a1 <= 1 and a2 <= 1 and a3 <= 1 then
					r,g,b = a1,a2,a3
				elseif (type(a1) == "number" or string.len(a1) == 2) and string.find(a1, "^%x*$") and (type(a2) == "number" or string.len(a2) == 2) and string.find(a2, "^%x*$") and (type(a3) == "number" or string.len(a3) == 2) and string.find(a3, "^%x*$") then
					r,g,b = tonumber(a1, 16) / 255, tonumber(a2, 16) / 255, tonumber(a3, 16) / 255
				end
			end
			if not r then
				print(string.format(options.error or IS_NOT_A_VALID_OPTION_FOR, table.concat(args, ' '), path), realOptions.cmdName or realOptions.name or self)
				print(string.format("|cffffff7f%s:|r %s {0-1} {0-1} {0-1}%s", USAGE, path, options.hasAlpha and " {0-1}" or ""))
				return
			end
			if passTable then
				if type(passTable.set) == "function" then
					passTable.set(passValue, r,g,b,a)
				else
					if type(handler[passTable.set]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, passTable.set)
					end
					handler[passTable.set](handler, passValue, r,g,b,a)
				end
			else
				if type(options.set) == "function" then
					options.set(r,g,b,a)
				else
					if type(handler[options.set]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.set)
					end
					handler[options.set](handler, r,g,b,a)
				end
			end

			local r,g,b,a
			if passTable then
				if type(passTable.get) == "function" then
					r,g,b,a = passTable.get(passValue)
				else
					if type(handler[passTable.get]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, passTable.get)
					end
					r,g,b,a = handler[passTable.get](handler, passValue)
				end
			else
				if type(options.get) == "function" then
					r,g,b,a = options.get()
				else
					if type(handler[options.get]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.get)
					end
					r,g,b,a = handler[options.get](handler)
				end
			end

			local s
			if type(r) == "number" and type(g) == "number" and type(b) == "number" then
				if options.hasAlpha and type(a) == "number" then
					s = string.format("|c%02x%02x%02x%02x%02x%02x%02x%02x|r", a*255, r*255, g*255, b*255, r*255, g*255, b*255, a*255)
				else
					s = string.format("|cff%02x%02x%02x%02x%02x%02x|r", r*255, g*255, b*255, r*255, g*255, b*255)
				end
			else
				s = NONE
			end
			print(string.format(options.message or IS_NOW_SET_TO, tostring(options.cmdName or options.name), s), realOptions.cmdName or realOptions.name or self)
		else
			local r,g,b,a
			if passTable then
				if type(passTable.get) == "function" then
					r,g,b,a = passTable.get(passValue)
				else
					if type(handler[passTable.get]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, passTable.get)
					end
					r,g,b,a = handler[passTable.get](handler, passValue)
				end
			else
				if type(options.get) == "function" then
					r,g,b,a = options.get()
				else
					if type(handler[options.get]) ~= "function" then
						AceConsole:error(OPTION_HANDLER_NOT_FOUND, options.get)
					end
					r,g,b,a = handler[options.get](handler)
				end
			end

			if not colorTable then
				colorTable = {}
				local t = colorTable

				if ColorPickerOkayButton then
					local ColorPickerOkayButton_OnClick = ColorPickerOkayButton:GetScript("OnClick")
					ColorPickerOkayButton:SetScript("OnClick", function()
						if ColorPickerOkayButton_OnClick then
							ColorPickerOkayButton_OnClick()
						end
						if t.active then
							ColorPickerFrame.cancelFunc = nil
							ColorPickerFrame.func = nil
							ColorPickerFrame.opacityFunc = nil
							local r,g,b,a
							if t.passValue then
								if type(t.get) == "function" then
									r,g,b,a = t.get(t.passValue)
								else
									if type(t.handler[t.get]) ~= "function" then
										AceConsole:error(OPTION_HANDLER_NOT_FOUND, t.get)
									end
									r,g,b,a = t.handler[t.get](t.handler, t.passValue)
								end
							else
								if type(t.get) == "function" then
									r,g,b,a = t.get()
								else
									if type(t.handler[t.get]) ~= "function" then
										AceConsole:error(OPTION_HANDLER_NOT_FOUND, t.get)
									end
									r,g,b,a = t.handler[t.get](t.handler)
								end
							end
							if r ~= t.r or g ~= t.g or b ~= t.b or (t.hasAlpha and a ~= t.a) then
								local s
								if type(r) == "number" and type(g) == "number" and type(b) == "number" then
									if t.hasAlpha and type(a) == "number" then
										s = string.format("|c%02x%02x%02x%02x%02x%02x%02x%02x|r", a*255, r*255, g*255, b*255, r*255, g*255, b*255, a*255)
									else
										s = string.format("|cff%02x%02x%02x%02x%02x%02x|r", r*255, g*255, b*255, r*255, g*255, b*255)
									end
								else
									s = NONE
								end
								print(string.format(t.message, tostring(t.name), s), t.realOptions.cmdName or t.realOptions.name or self)
							end
							for k,v in pairs(t) do
								t[k] = nil
							end
						end
					end)
				end
			else
				for k,v in pairs(colorTable) do
					colorTable[k] = nil
				end
			end

			if type(r) ~= "number" or type(g) ~= "number" or type(b) ~= "number" then
				r,g,b = 1, 1, 1
			end
			if type(a) ~= "number" then
				a = 1
			end
			local t = colorTable
			t.r = r
			t.g = g
			t.b = b
			if hasAlpha then
				t.a = a
			end
			t.realOptions = realOptions
			t.hasAlpha = options.hasAlpha
			t.handler = handler
			t.set = passTable and passTable.set or options.set
			t.get = passTable and passTable.get or options.get
			t.name = options.cmdName or options.name
			t.message = options.message or IS_NOW_SET_TO
			t.passValue = passValue
			t.active = true

			if not colorFunc then
				colorFunc = function()
					local r,g,b = ColorPickerFrame:GetColorRGB()
					if t.hasAlpha then
						local a = 1 - OpacitySliderFrame:GetValue()
						if type(t.set) == "function" then
							if t.passValue then
								t.set(t.passValue, r,g,b,a)
							else
								t.set(r,g,b,a)
							end
						else
							if type(t.handler[t.set]) ~= "function" then
								AceConsole:error(OPTION_HANDLER_NOT_FOUND, t.set)
							end
							if t.passValue then
								t.handler[t.set](t.handler, t.passValue, r,g,b,a)
							else
								t.handler[t.set](t.handler, r,g,b,a)
							end
						end
					else
						if type(t.set) == "function" then
							if t.passValue then
								t.set(t.passValue, r,g,b)
							else
								t.set(r,g,b)
							end
						else
							if type(t.handler[t.set]) ~= "function" then
								AceConsole:error(OPTION_HANDLER_NOT_FOUND, t.set)
							end
							if t.passValue then
								t.handler[t.set](t.handler, t.passValue, r,g,b)
							else
								t.handler[t.set](t.handler, r,g,b)
							end
						end
					end
				end
			end

			ColorPickerFrame.func = colorFunc
			ColorPickerFrame.hasOpacity = options.hasAlpha
			if options.hasAlpha then
				ColorPickerFrame.opacityFunc = ColorPickerFrame.func
				ColorPickerFrame.opacity = 1 - a
			end
			ColorPickerFrame:SetColorRGB(r,g,b)

			if not colorCancelFunc then
				colorCancelFunc = function()
					if t.hasAlpha then
						if type(t.set) == "function" then
							if t.passValue then
								t.set(t.passValue, t.r,t.g,t.b,t.a)
							else
								t.set(t.r,t.g,t.b,t.a)
							end
						else
							if type(t.handler[t.get]) ~= "function" then
								AceConsole:error(OPTION_HANDLER_NOT_FOUND, t.get)
							end
							if t.passValue then
								t.handler[t.set](t.handler, t.passValue, t.r,t.g,t.b,t.a)
							else
								t.handler[t.set](t.handler, t.r,t.g,t.b,t.a)
							end
						end
					else
						if type(t.set) == "function" then
							if t.passValue then
								t.set(t.passValue, t.r,t.g,t.b)
							else
								t.set(t.r,t.g,t.b)
							end
						else
							if type(t.handler[t.set]) ~= "function" then
								AceConsole:error(OPTION_HANDLER_NOT_FOUND, t.set)
							end
							if t.passValue then
								t.handler[t.set](t.handler, t.passValue, t.r,t.g,t.b)
							else
								t.handler[t.set](t.handler, t.r,t.g,t.b)
							end
						end
					end
					for k,v in pairs(t) do
						t[k] = nil
					end
					ColorPickerFrame.cancelFunc = nil
					ColorPickerFrame.func = nil
					ColorPickerFrame.opacityFunc = nil
				end
			end

			ColorPickerFrame.cancelFunc = colorCancelFunc

			ShowUIPanel(ColorPickerFrame)
		end
		return
	elseif kind == "group" then
		if table.getn(args) == 0 then
			printUsage(self, handler, realOptions, options, path, args)
		else
			-- invalid argument
			print(string.format(options.error or IS_NOT_A_VALID_OPTION_FOR, args[1], path), realOptions.cmdName or realOptions.name or self)
		end
		return
	end
	this = _G_this
	if Dewdrop then
		Dewdrop:Refresh(1)
		Dewdrop:Refresh(2)
		Dewdrop:Refresh(3)
		Dewdrop:Refresh(4)
		Dewdrop:Refresh(5)
	end
end

local external
function AceConsole:RegisterChatCommand(slashCommands, options, name)
	if type(slashCommands) ~= "table" and slashCommands ~= false then
		AceConsole:error("Bad argument #2 to `RegisterChatCommand' (expected table, got %s)", type(slashCommands))
	end
	if not slashCommands and type(name) ~= "string" then
		AceConsole:error("Bad argument #4 to `RegisterChatCommand' (expected string, got %s)", type(name))
	end
	if type(options) ~= "table" and type(options) ~= "function" and options ~= nil then
		AceConsole:error("Bad argument #3 to `RegisterChatCommand' (expected table, function, or nil, got %s)", type(options))
	end
	if name then
		if type(name) ~= "string" then
			AceConsole:error("Bad argument #4 to `RegisterChatCommand' (expected string or nil, got %s)", type(name))
		elseif not string.find(name, "^%w+$") or string.upper(name) ~= name or string.len(name) == 0 then
			AceConsole:error("Argument #4 must be an uppercase, letters-only string with at least 1 character")
		end
	end
	if slashCommands then
		if table.getn(slashCommands) == 0 then
			AceConsole:error("Argument #2 to `RegisterChatCommand' must include at least one string")
		end

		for k,v in pairs(slashCommands) do
			if type(k) ~= "number" then
				AceConsole:error("All keys in argument #2 to `RegisterChatCommand' must be numbers")
			end
			if type(v) ~= "string" then
				AceConsole:error("All values in argument #2 to `RegisterChatCommand' must be strings")
			elseif not string.find(v, "^/[A-Za-z][A-Za-z0-9_]*$") then
				AceConsole:error("All values in argument #2 to `RegisterChatCommand' must be in the form of \"/word\"")
			end
		end
	end

	if not options then
		options = {
			type = 'group',
			args = {},
			handler = self
		}
	end

	if type(options) == "table" then
		local err, position = validateOptions(options)
		if err then
			if position then
				AceConsole:error(position .. ": " .. err)
			else
				AceConsole:error(err)
			end
		end

		if not options.handler then
			options.handler = self
		end

		if options.handler == self and string.lower(options.type) == "group" and self.class then
			AceConsole:InjectAceOptionsTable(self, options)
		end
	end

	local chat
	if slashCommands then
		chat = slashCommands[1]
	else
		chat = _G["SLASH_"..name..1]
	end

	local handler
	if type(options) == "function" then
		handler = options
		for k,v in pairs(_G) do
			if handler == v then
				local k = k
				handler = function(msg)
					return _G[k](msg)
				end
			end
		end
	else
		function handler(msg)
			handlerFunc(self, chat, msg, options)
		end
	end

	if not _G.SlashCmdList then
		_G.SlashCmdList = {}
	end

	if not name then
		repeat
			name = string.char(math.random(26) + string.byte('A') - 1) .. string.char(math.random(26) + string.byte('A') - 1) .. string.char(math.random(26) + string.byte('A') - 1) .. string.char(math.random(26) + string.byte('A') - 1) .. string.char(math.random(26) + string.byte('A') - 1) .. string.char(math.random(26) + string.byte('A') - 1) .. string.char(math.random(26) + string.byte('A') - 1) .. string.char(math.random(26) + string.byte('A') - 1)
		until not _G.SlashCmdList[name]
	end

	if slashCommands then
		if _G.SlashCmdList[name] then
			local i = 0
			while true do
				i = i + 1
				if _G["SLASH_"..name..i] then
					_G["SLASH_"..name..i] = nil
				else
					break
				end
			end
		end

		local i = 0
		for _,command in ipairs(slashCommands) do
			i = i + 1
			_G["SLASH_"..name..i] = command
			if string.lower(command) ~= command then
				i = i + 1
				_G["SLASH_"..name..i] = string.lower(command)
			end
		end
	end
	_G.SlashCmdList[name] = handler
	if self ~= AceConsole and self.slashCommand == nil then
		self.slashCommand = chat
	end

	if not AceEvent and AceLibrary:HasInstance("AceEvent-2.0") then
		external(AceConsole, "AceEvent-2.0", AceLibrary("AceEvent-2.0"))
	end
	if AceEvent then
		if not AceConsole.nextAddon then
			AceConsole.nextAddon = {}
		end
		if type(options) == "table" then
			AceConsole.nextAddon[self] = options
			if not self.playerLogin then
				AceConsole:RegisterEvent("PLAYER_LOGIN", "PLAYER_LOGIN", true)
			end
		end
	end

	AceConsole.registry[name] = options
end

function AceConsole:InjectAceOptionsTable(handler, options)
	self:argCheck(handler, 2, "table")
	self:argCheck(options, 3, "table")
	if string.lower(options.type) ~= "group" then
		self:error('Cannot inject into options table argument #3 if its type is not "group"')
	end
	if options.handler ~= nil and options.handler ~= handler then
		self:error("Cannot inject into options table argument #3 if it has a different handler than argument #2")
	end
	options.handler = handler
	local class = handler.class
	if not class then
		self:error("Cannot retrieve AceOptions tables from a non-object argument #2")
	end
	while class and class ~= AceOO.Class do
		if type(class.GetAceOptionsDataTable) == "function" then
			local t = class:GetAceOptionsDataTable(handler)
			for k,v in pairs(t) do
				if type(options.args) ~= "table" then
					options.args = {}
				end
				if options.args[k] == nil then
					options.args[k] = v
				end
			end
		end
		local mixins = class.mixins
		if mixins then
			for mixin in pairs(mixins) do
				if type(mixin.GetAceOptionsDataTable) == "function" then
					local t = mixin:GetAceOptionsDataTable(handler)
					for k,v in pairs(t) do
						if type(options.args) ~= "table" then
							options.args = {}
						end
						if options.args[k] == nil then
							options.args[k] = v
						end
					end
				end
			end
		end
		class = class.super
	end
	return options
end

function AceConsole:PLAYER_LOGIN()
	self.playerLogin = true
	for addon, options in pairs(self.nextAddon) do
		local err, position = validateOptionsMethods(addon, options)
		if err then
			if position then
				error(tostring(addon) .. ": AceConsole: " .. position .. ": " .. err)
			else
				error(tostring(addon) .. ": AceConsole: " .. err)
			end
		end
		self.nextAddon[addon] = nil
	end

	self:RegisterChatCommand({ "/reload", "/rl", "/reloadui" }, ReloadUI, "RELOAD")

	local version = GetBuildInfo()
	if string.find(version, "^2%.") then
		-- 2.0.0
		self:RegisterChatCommand({ "/print" }, function(text)
			RunScript("local function func(...) local arg = {...}; for k = 1,select('#', ...) do arg[k] = tostring(arg[k]) end DEFAULT_CHAT_FRAME:AddMessage(table.concat(arg, ' ')) end func(" .. text .. ")")
		end, "PRINT")
	else
		self:RegisterChatCommand({ "/print" }, function(text)
			RunScript("local function func(...) for k = 1,table.getn(arg) do arg[k] = tostring(arg[k]) end DEFAULT_CHAT_FRAME:AddMessage(table.concat(arg, ' ')) end func(" .. text .. ")")
		end, "PRINT")
	end
end

function AceConsole:TabCompleteInfo(cmdpath)
	local _, _, cmd =  string.find(cmdpath, "(/%S+)")
	if not cmd then
		return
	end
	local path = string.sub(cmdpath, string.len(cmd) + 2)
	for name in pairs(SlashCmdList) do --global
		if AceConsole.registry[name] then
			local i = 0
			while true do
				i = i + 1
				local scmd = _G["SLASH_"..name..i]
				if not scmd then break end
				if cmd == scmd then
					return name, cmd, path
				end
			end
		end
	end
end

function external(self, major, instance)
	if major == "AceEvent-2.0" then
		if not AceEvent then
			AceEvent = instance

			AceEvent:embed(self)
		end
	elseif major == "AceTab-2.0" then
		instance:RegisterTabCompletion("AceConsole", "%/.*", function(t, cmdpath, pos)
			local ac = AceLibrary("AceConsole-2.0")
			local name, cmd, path = ac:TabCompleteInfo(string.sub(cmdpath, 1, pos))

			if not ac.registry[name] then
				return false
			else
				local validArgs = findTableLevel(ac, ac.registry[name], cmd, path or "")
				if validArgs.args then
					for arg in pairs(validArgs.args) do
						table.insert(t, arg)
					end
				end
			end
		end, function(u, matches, gcs, cmdpath)
			local ac = AceLibrary("AceConsole-2.0")
			local name, cmd, path = ac:TabCompleteInfo(cmdpath)
			if ac.registry[name] then
				local validArgs, path2, argwork = findTableLevel(ac, ac.registry[name], cmd, path)
				printUsage(ac, validArgs.handler, ac.registry[name], validArgs, path2, argwork, not gcs or gcs ~= "", gcs)
			end
		end)
	elseif major == "Dewdrop-2.0" then
		Dewdrop = instance
	end
end

local function activate(self, oldLib, oldDeactivate)
	AceConsole = self

	self.super.activate(self, oldLib, oldDeactivate)

	if oldLib then
		self.registry = oldLib.registry
		self.nextAddon = oldLib.nextAddon
	end
	if not self.registry then
		self.registry = {}
	else
		for name,options in pairs(self.registry) do
			self:RegisterChatCommand(false, options, name)
		end
	end

	if oldDeactivate then
		oldDeactivate(oldLib)
	end
end

AceLibrary:Register(AceConsole, MAJOR_VERSION, MINOR_VERSION, activate, nil, external)
AceConsole = AceLibrary(MAJOR_VERSION)
```

## lib/AceDB/AceDB-2.0.lua
```lua
--[[
Name: AceDB-2.0
Revision: $Rev: 14832 $
Developed by: The Ace Development Team (http://www.wowace.com/index.php/The_Ace_Development_Team)
Inspired By: Ace 1.x by Turan (turan@gryphon.com)
Website: http://www.wowace.com/
Documentation: http://www.wowace.com/index.php/AceDB-2.0
SVN: http://svn.wowace.com/root/trunk/Ace2/AceDB-2.0
Description: Mixin to allow for fast, clean, and featureful saved variable
             access.
Dependencies: AceLibrary, AceOO-2.0, AceEvent-2.0
]]

local MAJOR_VERSION = "AceDB-2.0"
local MINOR_VERSION = "$Revision: 14832 $"

if not AceLibrary then error(MAJOR_VERSION .. " requires AceLibrary") end
if not AceLibrary:IsNewVersion(MAJOR_VERSION, MINOR_VERSION) then return end

if not AceLibrary:HasInstance("AceOO-2.0") then error(MAJOR_VERSION .. " requires AceOO-2.0") end

local ACTIVE, ENABLED, STATE, TOGGLE_ACTIVE, MAP_ACTIVESUSPENDED, SET_PROFILE, SET_PROFILE_USAGE, PROFILE, PLAYER_OF_REALM, CHOOSE_PROFILE_DESC, CHOOSE_PROFILE_GUI, COPY_PROFILE_DESC, COPY_PROFILE_GUI, OTHER_PROFILE_DESC, OTHER_PROFILE_GUI, OTHER_PROFILE_USAGE, CHARACTER, REALM, CLASS

if GetLocale() == "deDE" then
	ACTIVE = "Aktiv"
	ENABLED = "Aktiviert"
	STATE = "Status"
	TOGGLE_ACTIVE = "Stoppt/Aktiviert dieses Addon."
	MAP_ACTIVESUSPENDED = { [true] = "|cff00ff00Aktiv|r", [false] = "|cffff0000Gestoppt|r" }
	SET_PROFILE = "Setzt das Profil f\195\188r dieses Addon."
	SET_PROFILE_USAGE = "{Charakter || Klasse || Realm || <Profilname>}"
	PROFILE = "Profil"
	PLAYER_OF_REALM = "%s von %s"
	CHOOSE_PROFILE_DESC = "W\195\164hle ein Profil."
	CHOOSE_PROFILE_GUI = "W\195\164hle"
	COPY_PROFILE_DESC = "Kopiert Einstellungen von einem anderem Profil."
	COPY_PROFILE_GUI = "Kopiere von"
	OTHER_PROFILE_DESC = "W\195\164hle ein anderes Profil."
	OTHER_PROFILE_GUI = "Anderes"
	OTHER_PROFILE_USAGE = "<Profilname>"

	CHARACTER = "Charakter: "
	REALM = "Realm: "
	CLASS = "Klasse: "
elseif GetLocale() == "frFR" then
	ACTIVE = "Actif"
	ENABLED = "Activ\195\169"
	STATE = "Etat"
	TOGGLE_ACTIVE = "Suspend/active cet addon."
	MAP_ACTIVESUSPENDED = { [true] = "|cff00ff00Actif|r", [false] = "|cffff0000Suspendu|r" }
	SET_PROFILE = "S\195\169lectionne le profil pour cet addon."
	SET_PROFILE_USAGE = "{perso || classe || royaume || <nom de profil>}"
	PROFILE = "Profil"
	PLAYER_OF_REALM = "%s de %s"
	CHOOSE_PROFILE_DESC = "Choisissez un profil."
	CHOOSE_PROFILE_GUI = "Choix"
	COPY_PROFILE_DESC = "Copier les param\195\168tres d'un autre profil."
	COPY_PROFILE_GUI = "Copier \195\160 partir de"
	OTHER_PROFILE_DESC = "Choisissez un autre profil."
	OTHER_PROFILE_GUI = "Autre"
	OTHER_PROFILE_USAGE = "<nom de profil>"

	CHARACTER = "Personnage: "
	REALM = "Royaume: "
	CLASS = "Classe: "
elseif GetLocale() == "koKR" then
	ACTIVE = "활성화"
	ENABLED = "활성화"
	STATE = "상태"
	TOGGLE_ACTIVE = "이 애드온 중지/계속 실행"
	MAP_ACTIVESUSPENDED = { [true] = "|cff00ff00활성화|r", [false] = "|cffff0000중지됨|r" }
	SET_PROFILE = "이 애드온에 프로필 설정"
	SET_PROFILE_USAGE = "{캐릭터명 || 직업 || 서버명 || <프로필명>}"
	PROFILE = "프로필"
	PLAYER_OF_REALM = "%s (%s 서버)"
	CHOOSE_PROFILE_DESC = "프로파일을 선택합니다."
	CHOOSE_PROFILE_GUI = "선택"
	COPY_PROFILE_DESC = "다른 프로파일에서 설정을 복사합니다."
	COPY_PROFILE_GUI = "복사"
	OTHER_PROFILE_DESC = "다른 프로파일을 선택합니다."
	OTHER_PROFILE_GUI = "기타"
	OTHER_PROFILE_USAGE = "<프로파일명>"

	CHARACTER = "캐릭터: "
	REALM = "서버: "
	CLASS = "직업: "
elseif GetLocale() == "zhTW" then
	ACTIVE = "啟動"
	ENABLED = "啟用"
	STATE = "狀態"
	TOGGLE_ACTIVE = "暫停/重啟這個插件。"
	MAP_ACTIVESUSPENDED = { [true] = "|cff00ff00啟動|r", [false] = "|cffff0000已暫停|r" }
	SET_PROFILE = "設定這插件的記錄檔。"
	SET_PROFILE_USAGE = "{角色 || 聯業 || 伺服器 || <記錄檔名稱>}"
	PROFILE = "記錄檔"
	PLAYER_OF_REALM = "%s 於 %s"
	CHOOSE_PROFILE_DESC = "選擇一個記錄檔"
	CHOOSE_PROFILE_GUI = "選擇"
	COPY_PROFILE_DESC = "由其他記錄檔複製設定。"
	COPY_PROFILE_GUI = "複製由"
	OTHER_PROFILE_DESC = "選擇其他記錄檔。"
	OTHER_PROFILE_GUI = "其他"
	OTHER_PROFILE_USAGE = "<記錄檔名稱>"

	CHARACTER = "角色："
	REALM = "伺服器："
	CLASS = "聯業："
elseif GetLocale() == "zhCN" then
	ACTIVE = "\230\156\137\230\149\136"
	ENABLED = "\229\144\175\231\148\168"
	STATE = "\231\138\182\230\128\129"
	TOGGLE_ACTIVE = "\230\154\130\229\129\156/\230\129\162\229\164\141 \230\173\164\230\143\146\228\187\182."
	MAP_ACTIVESUSPENDED = { [true] = "|cff00ff00\230\156\137\230\149\136|r", [false] = "|cffff0000\230\154\130\229\129\156|r" }
	SET_PROFILE = "\232\174\190\231\189\174\233\133\141\231\189\174\230\150\135\228\187\182\228\184\186\232\191\153\230\143\146\228\187\182."
	SET_PROFILE_USAGE = "{\229\173\151\231\172\166 || \233\128\137\228\187\182\231\177\187 || \229\159\159 || <\233\133\141\231\189\174\230\150\135\228\187\182\229\144\141\229\173\151>}"
	PROFILE = "\233\133\141\231\189\174\230\150\135\228\187\182"
	PLAYER_OF_REALM = "%s \231\154\132 %s"
	CHOOSE_PROFILE_DESC = "\233\128\137\230\139\169\233\133\141\231\189\174\230\150\135\228\187\182."
	CHOOSE_PROFILE_GUI = "\233\128\137\230\139\169"
	COPY_PROFILE_DESC = "\229\164\141\229\136\182\232\174\190\231\189\174\228\187\142\229\143\166\228\184\128\228\184\170\233\133\141\231\189\174\230\150\135\228\187\182."
	COPY_PROFILE_GUI = "\229\164\141\229\136\182\228\187\142"
	OTHER_PROFILE_DESC = "\233\128\137\230\139\169\229\143\166\228\184\128\228\184\170\233\133\141\231\189\174\230\150\135\228\187\182."
	OTHER_PROFILE_GUI = "\229\133\182\228\187\150"
	OTHER_PROFILE_USAGE = "<\233\133\141\231\189\174\230\150\135\228\187\182\229\144\141\229\173\151>"

	CHARACTER = "\229\173\151\231\172\166: "
	REALM = "\229\159\159: "
	CLASS = "\233\128\137\228\187\182\231\177\187: "
else -- enUS
	ACTIVE = "Active"
	ENABLED = "Enabled"
	STATE = "State"
	TOGGLE_ACTIVE = "Suspend/resume this addon."
	MAP_ACTIVESUSPENDED = { [true] = "|cff00ff00Active|r", [false] = "|cffff0000Suspended|r" }
	SET_PROFILE = "Set profile for this addon."
	SET_PROFILE_USAGE = "{char || class || realm || <profile name>}"
	PROFILE = "Profile"
	PLAYER_OF_REALM = "%s of %s"
	CHOOSE_PROFILE_DESC = "Choose a profile."
	CHOOSE_PROFILE_GUI = "Choose"
	COPY_PROFILE_DESC = "Copy settings from another profile."
	COPY_PROFILE_GUI = "Copy from"
	OTHER_PROFILE_DESC = "Choose another profile."
	OTHER_PROFILE_GUI = "Other"
	OTHER_PROFILE_USAGE = "<profile name>"

	CHARACTER = "Character: "
	REALM = "Realm: "
	CLASS = "Class: "
end

local AceOO = AceLibrary("AceOO-2.0")
local AceEvent
local Mixin = AceOO.Mixin
local AceDB = Mixin {
						"RegisterDB",
						"RegisterDefaults",
						"ResetDB",
						"SetProfile",
						"GetProfile",
						"ToggleActive",
						"IsActive",
						"AcquireDBNamespace",
					}
local Dewdrop = AceLibrary:HasInstance("Dewdrop-2.0") and AceLibrary("Dewdrop-2.0")

local _G = getfenv(0)

local table_setn
do
	local version = GetBuildInfo()
	if string.find(version, "^2%.") then
		-- 2.0.0
		table_setn = function() end
	else
		table_setn = table.setn
	end
end

local function inheritDefaults(t, defaults)
	if not defaults then
		return t
	end
	for k,v in pairs(defaults) do
		if k == "*" then
			local v = v
			if type(v) == "table" then
				setmetatable(t, {
					__index = function(self, key)
						if key == nil then
							return nil
						end
						self[key] = {}
						inheritDefaults(self[key], v)
						return self[key]
					end
				} )
			else
				setmetatable(t, {
					__index = function(self, key)
						if key == nil then
							return nil
						end
						self[key] = v
						return self[key]
					end
				} )
			end
			for key in pairs(t) do
				if (defaults[key] == nil or key == "*") and type(t[key]) == "table" then
					inheritDefaults(t[key], v)
				end
			end
		else
			if type(v) == "table" then
				if type(t[k]) ~= "table" then
					t[k] = {}
				end
				inheritDefaults(t[k], v)
			elseif t[k] == nil then
				t[k] = v
			end
		end
	end
	return t
end

local _,race = UnitRace("player")
local faction
if race == "Orc" or race == "Scourge" or race == "Troll" or race == "Tauren" then
	faction = FACTION_HORDE
else
	faction = FACTION_ALLIANCE
end
local charID = string.format(PLAYER_OF_REALM, UnitName("player"), (string.gsub(GetRealmName(), "^%s*(.-)%s*$", "%1")))
local realm = string.gsub(GetRealmName(), "^%s*(.-)%s*$", "%1")
local realmID = realm .. " - " .. faction
local classID = UnitClass("player")

AceDB.CHAR_ID = charID
AceDB.REALM_ID = realmID
AceDB.CLASS_ID = classID

AceDB.FACTION = faction
AceDB.REALM = realm
AceDB.NAME = UnitName("player")

local new, del
do
	local list = setmetatable({}, {__mode="k"})
	function new()
		local t = next(list)
		if t then
			list[t] = nil
			return t
		else
			return {}
		end
	end

	function del(t)
		setmetatable(t, nil)
		for k in pairs(t) do
			t[k] = nil
		end
		table_setn(t, 0)
		list[t] = true
	end
end

local caseInsensitive_mt = {
	__index = function(self, key)
		if type(key) ~= "string" then
			return nil
		end
		local lowerKey = string.lower(key)
		for k,v in pairs(self) do
			if string.lower(k) == lowerKey then
				return self[k]
			end
		end
	end,
	__newindex = function(self, key, value)
		if type(key) ~= "string" then
			return error("table index is nil", 2)
		end
		local lowerKey = string.lower(key)
		for k in pairs(self) do
			if string.lower(k) == lowerKey then
				rawset(self, k, nil)
				rawset(self, key, value)
				return
			end
		end
		rawset(self, key, value)
	end
}

local db_mt = { __index = function(db, key)
	if key == "char" then
		if db.charName then
			if type(_G[db.charName]) ~= "table" then
				_G[db.charName] = {}
			end
			if type(_G[db.charName].global) ~= "table" then
				_G[db.charName].global = {}
			end
			rawset(db, 'char', _G[db.charName].global)
		else
			if type(db.raw.chars) ~= "table" then
				db.raw.chars = {}
			end
			local id = charID
			if type(db.raw.chars[id]) ~= "table" then
				db.raw.chars[id] = {}
			end
			rawset(db, 'char', db.raw.chars[id])
		end
		if db.defaults and db.defaults.char then
			inheritDefaults(db.char, db.defaults.char)
		end
		return db.char
	elseif key == "realm" then
		if type(db.raw.realms) ~= "table" then
			db.raw.realms = {}
		end
		local id = realmID
		if type(db.raw.realms[id]) ~= "table" then
			db.raw.realms[id] = {}
		end
		rawset(db, 'realm', db.raw.realms[id])
		if db.defaults and db.defaults.realm then
			inheritDefaults(db.realm, db.defaults.realm)
		end
		return db.realm
	elseif key == "account" then
		if type(db.raw.account) ~= "table" then
			db.raw.account = {}
		end
		rawset(db, 'account', db.raw.account)
		if db.defaults and db.defaults.account then
			inheritDefaults(db.account, db.defaults.account)
		end
		return db.account
	elseif key == "class" then
		if type(db.raw.classes) ~= "table" then
			db.raw.classes = {}
		end
		local id = classID
		if type(db.raw.classes[id]) ~= "table" then
			db.raw.classes[id] = {}
		end
		rawset(db, 'class', db.raw.classes[id])
		if db.defaults and db.defaults.class then
			inheritDefaults(db.class, db.defaults.class)
		end
		return db.class
	elseif key == "profile" then
		if type(db.raw.profiles) ~= "table" then
			db.raw.profiles = setmetatable({}, caseInsensitive_mt)
		else
			setmetatable(db.raw.profiles, caseInsensitive_mt)
		end
		local id = db.raw.currentProfile[charID]
		if id == "char" then
			id = "char/" .. charID
		elseif id == "class" then
			id = "class/" .. classID
		elseif id == "realm" then
			id = "realm/" .. realmID
		end
		if type(db.raw.profiles[id]) ~= "table" then
			db.raw.profiles[id] = {}
		end
		rawset(db, 'profile', db.raw.profiles[id])
		if db.defaults and db.defaults.profile then
			inheritDefaults(db.profile, db.defaults.profile)
		end
		return db.profile
	elseif key == "raw" or key == "defaults" or key == "name" or key == "charName" or key == "namespaces" then
		return nil
	end
	error(string.format('Cannot access key %q in db table. You may want to use db.profile[%q]', tostring(key), tostring(key)), 2)
end, __newindex = function(db, key, value)
	error(string.format('Cannot access key %q in db table. You may want to use db.profile[%q]', tostring(key), tostring(key)), 2)
end }

local CrawlForSerialization
local CrawlForDeserialization

local function SerializeObject(o)
	local t = { o:Serialize() }
	t[0] = o.class:GetLibraryVersion()
	CrawlForSerialization(t)
	return t
end

local function DeserializeObject(t)
	CrawlForDeserialization(t)
	local className = t[0]
	for i = 20, 1, -1 do
		if t[i] then
			table_setn(t, i)
			break
		end
	end
	local o = AceLibrary(className):Deserialize(unpack(t))
	table_setn(t, 0)
	return o
end

local function IsSerializable(t)
	return AceOO.inherits(t, AceOO.Class) and t.class and type(t.class.Deserialize) == "function" and type(t.Serialize) == "function" and type(t.class.GetLibraryVersion) == "function"
end

function CrawlForSerialization(t)
	local tmp = new()
	for k,v in pairs(t) do
		tmp[k] = v
	end
	for k,v in pairs(tmp) do
		if type(v) == "table" and type(v[0]) ~= "userdata" then
			if IsSerializable(v) then
				v = SerializeObject(v)
				t[k] = v
			else
				CrawlForSerialization(v)
			end
		end
		if type(k) == "table" and type(k[0]) ~= "userdata" then
			if IsSerializable(k) then
				t[k] = nil
				t[SerializeObject(k)] = v
			else
				CrawlForSerialization(k)
			end
		end
		tmp[k] = nil
		k = nil
	end
	tmp = del(tmp)
end

local function IsDeserializable(t)
	return type(t[0]) == "string" and AceLibrary:HasInstance(t[0])
end

function CrawlForDeserialization(t)
	local tmp = new()
	for k,v in pairs(t) do
		tmp[k] = v
	end
	for k,v in pairs(tmp) do
		if type(v) == "table" then
			if IsDeserializable(v) then
				t[k] = DeserializeObject(v)
				del(v)
				v = t[k]
			elseif type(v[0]) ~= "userdata" then
				CrawlForDeserialization(v)
			end
		end
		if type(k) == "table" then
			if IsDeserializable(k) then
				t[k] = nil
				t[DeserializeObject(k)] = v
				del(k)
			elseif type(k[0]) ~= "userdata" then
				CrawlForDeserialization(k)
			end
		end
		tmp[k] = nil
		k = nil
	end
	tmp = del(tmp)
end

local namespace_mt = { __index = function(namespace, key)
	local db = namespace.db
	local name = namespace.name
	if key == "char" then
		if db.charName then
			if type(_G[db.charName]) ~= "table" then
				_G[db.charName] = {}
			end
			if type(_G[db.charName].namespaces) ~= "table" then
				_G[db.charName].namespaces = {}
			end
			if type(_G[db.charName].namespaces[name]) ~= "table" then
				_G[db.charName].namespaces[name] = {}
			end
			rawset(namespace, 'char', _G[db.charName].namespaces[name])
		else
			if type(db.raw.namespaces) ~= "table" then
				db.raw.namespaces = {}
			end
			if type(db.raw.namespaces[name]) ~= "table" then
				db.raw.namespaces[name] = {}
			end
			if type(db.raw.namespaces[name].chars) ~= "table" then
				db.raw.namespaces[name].chars = {}
			end
			local id = charID
			if type(db.raw.namespaces[name].chars[id]) ~= "table" then
				db.raw.namespaces[name].chars[id] = {}
			end
			rawset(namespace, 'char', db.raw.namespaces[name].chars[id])
		end
		if namespace.defaults and namespace.defaults.char then
			inheritDefaults(namespace.char, namespace.defaults.char)
		end
		return namespace.char
	elseif key == "realm" then
		if type(db.raw.namespaces) ~= "table" then
			db.raw.namespaces = {}
		end
		if type(db.raw.namespaces[name]) ~= "table" then
			db.raw.namespaces[name] = {}
		end
		if type(db.raw.namespaces[name].realms) ~= "table" then
			db.raw.namespaces[name].realms = {}
		end
		local id = realmID
		if type(db.raw.namespaces[name].realms[id]) ~= "table" then
			db.raw.namespaces[name].realms[id] = {}
		end
		rawset(namespace, 'realm', db.raw.namespaces[name].realms[id])
		if namespace.defaults and namespace.defaults.realm then
			inheritDefaults(namespace.realm, namespace.defaults.realm)
		end
		return namespace.realm
	elseif key == "account" then
		if type(db.raw.namespaces) ~= "table" then
			db.raw.namespaces = {}
		end
		if type(db.raw.namespaces[name]) ~= "table" then
			db.raw.namespaces[name] = {}
		end
		if type(db.raw.namespaces[name].account) ~= "table" then
			db.raw.namespaces[name].account = {}
		end
		rawset(namespace, 'account', db.raw.namespaces[name].account)
		if namespace.defaults and namespace.defaults.account then
			inheritDefaults(namespace.account, namespace.defaults.account)
		end
		return namespace.account
	elseif key == "class" then
		if type(db.raw.namespaces) ~= "table" then
			db.raw.namespaces = {}
		end
		if type(db.raw.namespaces[name]) ~= "table" then
			db.raw.namespaces[name] = {}
		end
		if type(db.raw.namespaces[name].classes) ~= "table" then
			db.raw.namespaces[name].classes = {}
		end
		local id = classID
		if type(db.raw.namespaces[name].classes[id]) ~= "table" then
			db.raw.namespaces[name].classes[id] = {}
		end
		rawset(namespace, 'class', db.raw.namespaces[name].classes[id])
		if namespace.defaults and namespace.defaults.class then
			inheritDefaults(namespace.class, namespace.defaults.class)
		end
		return namespace.class
	elseif key == "profile" then
		if type(db.raw.namespaces) ~= "table" then
			db.raw.namespaces = {}
		end
		if type(db.raw.namespaces[name]) ~= "table" then
			db.raw.namespaces[name] = {}
		end
		if type(db.raw.namespaces[name].profiles) ~= "table" then
			db.raw.namespaces[name].profiles = setmetatable({}, caseInsensitive_mt)
		else
			setmetatable(db.raw.namespaces[name].profiles, caseInsensitive_mt)
		end
		local id = db.raw.currentProfile[charID]
		if id == "char" then
			id = "char/" .. charID
		elseif id == "class" then
			id = "class/" .. classID
		elseif id == "realm" then
			id = "realm/" .. realmID
		end
		if type(db.raw.namespaces[name].profiles[id]) ~= "table" then
			db.raw.namespaces[name].profiles[id] = {}
		end
		rawset(namespace, 'profile', db.raw.namespaces[name].profiles[id])
		if namespace.defaults and namespace.defaults.profile then
			inheritDefaults(namespace.profile, namespace.defaults.profile)
		end
		return namespace.profile
	elseif key == "defaults" or key == "name" or key == "db" then
		return nil
	end
	error(string.format('Cannot access key %q in db table. You may want to use db.profile[%q]', tostring(key), tostring(key)), 2)
end, __newindex = function(db, key, value)
	error(string.format('Cannot access key %q in db table. You may want to use db.profile[%q]', tostring(key), tostring(key)), 2)
end }

function AceDB:InitializeDB(addonName)
	local db = self.db

	if not db then
		if addonName then
			AceDB.addonsLoaded[addonName] = true
		end
		return
	end

	if db.raw then
		-- someone manually initialized
		return
	end

	if type(_G[db.name]) ~= "table" then
		_G[db.name] = {}
	else
		CrawlForDeserialization(_G[db.name])
	end
	if type(_G[db.charName]) == "table" then
		CrawlForDeserialization(_G[db.charName])
	end
	rawset(db, 'raw', _G[db.name])
	if not db.raw.currentProfile then
		db.raw.currentProfile = {}
	end
	if not db.raw.currentProfile[charID] then
		db.raw.currentProfile[charID] = "Default"
	end
	if db.raw.disabled then
		setmetatable(db.raw.disabled, caseInsensitive_mt)
	end
	if self['acedb-profile-copylist'] then
		local t = self['acedb-profile-copylist']
		for k,v in pairs(t) do
			t[k] = nil
		end
		if db.raw.profiles then
			for k in pairs(db.raw.profiles) do
				if string.find(k, '^char/') then
					local name = string.sub(k, 6)
					if name ~= charID then
						t[k] =  CHARACTER .. name
					end
				elseif string.find(k, '^realm/') then
					local name = string.sub(k, 7)
					if name ~= realmID then
						t[k] =  REALM .. name
					end
				elseif string.find(k, '^class/') then
					local name = string.sub(k, 7)
					if name ~= classID then
						t[k] =  CLASS .. name
					end
				end
			end
		end
	end
	if self['acedb-profile-list'] then
		local t = self['acedb-profile-list']
		for k,v in pairs(t) do
			t[k] = nil
		end
		t.char = CHARACTER .. charID
		t.realm = REALM .. realmID
		t.class = CLASS .. classID
		t.Default = "Default"
		if db.raw.profiles then
			for k in pairs(db.raw.profiles) do
				if not string.find(k, '^char/') and not string.find(k, '^realm/') and not string.find(k, '^class/') then
					t[k] = k
				end
			end
		end
	end
	setmetatable(db, db_mt)
end

function AceDB:OnEmbedInitialize(target, name)
	if name then
		self:ADDON_LOADED(name)
	end
	self.InitializeDB(target, name)
end

function AceDB:RegisterDB(name, charName)
	AceDB:argCheck(name, 2, "string")
	AceDB:argCheck(charName, 3, "string", "nil")
	if self.db then
		AceDB:error("Cannot call \"RegisterDB\" if self.db is set.")
	end
	local stack = debugstack()
	local addonName = string.gsub(stack, ".-\n.-\\AddOns\\(.-)\\.*", "%1")
	self.db = {
		name = name,
		charName = charName
	}
	if AceDB.addonsLoaded[addonName] then
		AceDB.InitializeDB(self, addonName)
	else
		AceDB.addonsToBeInitialized[self] = addonName
	end
	AceDB.registry[self] = true
end

function AceDB:RegisterDefaults(kind, defaults, a3)
	local name
	if a3 then
		name, kind, defaults = kind, defaults, a3
		AceDB:argCheck(name, 2, "string")
		AceDB:argCheck(kind, 3, "string")
		AceDB:argCheck(defaults, 4, "table")
		if kind ~= "char" and kind ~= "class" and kind ~= "profile" and kind ~= "account" and kind ~= "realm" then
			AceDB:error("Bad argument #3 to `RegisterDefaults' (\"char\", \"class\", \"profile\", \"account\", or \"realm\" expected, got %q)", kind)
		end
	else
		AceDB:argCheck(kind, 2, "string")
		AceDB:argCheck(defaults, 3, "table")
		if kind ~= "char" and kind ~= "class" and kind ~= "profile" and kind ~= "account" and kind ~= "realm" then
			AceDB:error("Bad argument #2 to `RegisterDefaults' (\"char\", \"class\", \"profile\", \"account\", or \"realm\" expected, got %q)", kind)
		end
	end
	if type(self.db) ~= "table" or type(self.db.name) ~= "string" then
		AceDB:error("Cannot call \"RegisterDefaults\" unless \"RegisterDB\" has been previously called.")
	end
	local db
	if name then
		local namespace = self:AcquireDBNamespace(name)
		if namespace.defaults and namespace.defaults[kind] then
			AceDB:error("\"RegisterDefaults\" has already been called for %q::%q.", name, kind)
		end
		db = namespace
	else
		if self.db.defaults and self.db.defaults[kind] then
			AceDB:error("\"RegisterDefaults\" has already been called for %q.", kind)
		end
		db = self.db
	end
	if not db.defaults then
		rawset(db, 'defaults', {})
	end
	db.defaults[kind] = defaults
	if rawget(db, kind) then
		inheritDefaults(db[kind], defaults)
	end
end

function AceDB:ResetDB(kind)
	AceDB:argCheck(kind, 2, "nil", "string")
	if not self.db or not self.db.raw then
		AceDB:error("Cannot call \"ResetDB\" before \"RegisterDB\" has been called and before \"ADDON_LOADED\" has been fired.")
	end
	local db = self.db
	if kind == nil then
		if db.charName then
			_G[db.charName] = nil
		end
		_G[db.name] = nil
		rawset(db, 'raw', nil)
		AceDB.InitializeDB(self)
		if db.namespaces then
			for name,v in pairs(db.namespaces) do
				rawset(v, 'account', nil)
				rawset(v, 'char', nil)
				rawset(v, 'class', nil)
				rawset(v, 'profile', nil)
				rawset(v, 'realm', nil)
			end
		end
	elseif kind == "account" then
		db.raw.account = nil
		rawset(db, 'account', nil)
		if db.namespaces then
			for name,v in pairs(db.namespaces) do
				rawset(v, 'account', nil)
			end
		end
	elseif kind == "char" then
		if db.charName then
			_G[db.charName] = nil
		else
			if db.raw.chars then
				db.raw.chars[charID] = nil
			end
			if db.raw.namespaces then
				for name,v in pairs(db.raw.namespaces) do
					if v.chars then
						v.chars[charID] = nil
					end
				end
			end
		end
		rawset(db, 'char', nil)
		if db.namespaces then
			for name,v in pairs(db.namespaces) do
				rawset(v, 'char', nil)
			end
		end
	elseif kind == "realm" then
		if db.raw.realms then
			db.raw.realms[realmID] = nil
		end
		rawset(db, 'realm', nil)
		if db.raw.namespaces then
			for name,v in pairs(db.raw.namespaces) do
				if v.realms then
					v.realms[realmID] = nil
				end
			end
		end
		if db.namespaces then
			for name,v in pairs(db.namespaces) do
				rawset(v, 'realm', nil)
			end
		end
	elseif kind == "class" then
		if db.raw.realms then
			db.raw.realms[classID] = nil
		end
		rawset(db, 'class', nil)
		if db.raw.namespaces then
			for name,v in pairs(db.raw.namespaces) do
				if v.classes then
					v.classes[classID] = nil
				end
			end
		end
		if db.namespaces then
			for name,v in pairs(db.namespaces) do
				rawset(v, 'class', nil)
			end
		end
	elseif kind == "profile" then
		local id = db.raw.currentProfile and db.raw.currentProfile[charID] or "Default"
		if id == "char" then
			id = "char/" .. charID
		elseif id == "class" then
			id = "class/" .. classID
		elseif id == "realm" then
			id = "realm/" .. realmID
		end
		if db.raw.profiles then
			db.raw.profiles[id] = nil
		end
		rawset(db, 'profile', nil)
		if db.raw.namespaces then
			for name,v in pairs(db.raw.namespaces) do
				if v.profiles then
					v.profiles[id] = nil
				end
			end
		end
		if db.namespaces then
			for name,v in pairs(db.namespaces) do
				rawset(v, 'profile', nil)
			end
		end
	end
end

local function cleanDefaults(t, defaults)
	if defaults then
		for k,v in pairs(defaults) do
			if k == "*" then
				if type(v) == "table" then
					for k in pairs(t) do
						if (defaults[k] == nil or k == "*") and type(t[k]) == "table" then
							if cleanDefaults(t[k], v) then
								t[k] = nil
							end
						end
					end
				else
					for k in pairs(t) do
						if (defaults[k] == nil or k == "*") and t[k] == v then
							t[k] = nil
						end
					end
				end
			else
				if type(v) == "table" then
					if type(t[k]) == "table" then
						if cleanDefaults(t[k], v) then
							t[k] = nil
						end
					end
				elseif t[k] == v then
					t[k] = nil
				end
			end
		end
	end
	return t and not next(t)
end

function AceDB:GetProfile()
	if not self.db or not self.db.raw then
		return nil
	end
	if not self.db.raw.currentProfile then
		self.db.raw.currentProfile = {}
	end
	if not self.db.raw.currentProfile[charID] then
		self.db.raw.currentProfile[charID] = "Default"
	end
	local profile = self.db.raw.currentProfile[charID]
	if profile == "char" then
		return "char", "char/" .. charID
	elseif profile == "class" then
		return "class", "class/" .. classID
	elseif profile == "realm" then
		return "realm", "realm/" .. realmID
	end
	return profile, profile
end

local function copyTable(to, from)
	setmetatable(to, nil)
	for k,v in pairs(from) do
		if type(k) == "table" then
			k = copyTable({}, k)
		end
		if type(v) == "table" then
			v = copyTable({}, v)
		end
		to[k] = v
	end
	table_setn(to, table.getn(from))
	setmetatable(to, from)
	return to
end

function AceDB:SetProfile(name, copyFrom)
	AceDB:argCheck(name, 2, "string")
	AceDB:argCheck(copyFrom, 3, "string", "nil")
	if not self.db or not self.db.raw then
		AceDB:error("Cannot call \"SetProfile\" before \"RegisterDB\" has been called and before \"ADDON_LOADED\" has been fired.")
	end
	local db = self.db
	local copy = false
	local lowerName = string.lower(name)
	local lowerCopyFrom = copyFrom and string.lower(copyFrom)
	if string.sub(lowerName, 1, 5) == "char/" or string.sub(lowerName, 1, 6) == "realm/" or string.sub(lowerName, 1, 6) == "class/" then
		if string.sub(lowerName, 1, 5) == "char/" then
			name, copyFrom = "char", name
		else
			name, copyFrom = string.sub(lowerName, 1, 5), name
		end
		lowerName = string.lower(name)
		lowerCopyFrom = string.lower(copyFrom)
	end
	if copyFrom then
		if string.sub(lowerCopyFrom, 1, 5) == "char/" then
			AceDB:assert(lowerName == "char", "If argument #3 starts with `char/', argument #2 must be `char'")
		elseif string.sub(lowerCopyFrom, 1, 6) == "realm/" then
			AceDB:assert(lowerName == "realm", "If argument #3 starts with `realm/', argument #2 must be `realm'")
		elseif string.sub(lowerCopyFrom, 1, 6) == "class/" then
			AceDB:assert(lowerName == "class", "If argument #3 starts with `class/', argument #2 must be `class'")
		else
			AceDB:assert(lowerName ~= "char" and lowerName ~= "realm" and lowerName ~= "class", "If argument #3 does not start with a special prefix, that prefix cannot be copied to.")
		end
		if not db.raw.profiles or not db.raw.profiles[copyFrom] then
			AceDB:error("Cannot copy profile %q, it does not exist.", copyFrom)
		elseif (string.sub(lowerName, 1, 5) == "char/" and string.sub(lowerName, 6) == string.lower(charID)) or (string.sub(lowerName, 1, 6) == "realm/" and string.sub(lowerName, 7) == string.lower(realmID)) or (string.sub(lowerName, 1, 6) == "class/" and string.sub(lowerName, 7) == string.lower(classID)) then
			AceDB:error("Cannot copy profile %q, it is currently in use.", name)
		end
	end
	local oldName = db.raw.currentProfile[charID]
	if string.lower(oldName) == string.lower(name) then
		return
	end
	local current = self.class
	while current and current ~= AceOO.Class do
		if current.mixins then
			for mixin in pairs(current.mixins) do
				if type(mixin.OnEmbedProfileDisable) == "function" then
					mixin:OnEmbedProfileDisable(self)
				end
			end
		end
		current = current.super
	end
	if type(self.OnProfileDisable) == "function" then
		self:OnProfileDisable()
	end
	local oldProfileData = db.profile
	local realName = name
	if lowerName == "char" then
		realName = name .. "/" .. charID
	elseif lowerName == "realm/" then
		realName = name .. "/" .. realmID
	elseif lowerName == "class/" then
		realName = name .. "/" .. classID
	end
	local active = self:IsActive()
	db.raw.currentProfile[charID] = name
	rawset(db, 'profile', nil)
	if copyFrom then
		for k,v in pairs(db.profile) do
			db.profile[k] = nil
		end
		copyTable(db.profile, db.raw.profiles[copyFrom])
		inheritDefaults(db.profile, db.defaults and db.defaults.profile)
	end
	local current = self.class
	while current and current ~= AceOO.Class do
		if current.mixins then
			for mixin in pairs(current.mixins) do
				if type(mixin.OnEmbedProfileEnable) == "function" then
					mixin:OnEmbedProfileEnable(self, oldName, oldProfileData, copyFrom)
				end
			end
		end
		current = current.super
	end
	if type(self.OnProfileEnable) == "function" then
		self:OnProfileEnable(oldName, oldProfileData, copyFrom)
	end
	if cleanDefaults(oldProfileData, db.defaults and db.defaults.profile) then
		db.raw.profiles[oldName] = nil
		if not next(db.raw.profiles) then
			db.raw.profiles = nil
		end
	end
	local newactive = self:IsActive()
	if active ~= newactive then
		if AceOO.inherits(self, "AceAddon-2.0") then
			local AceAddon = AceLibrary("AceAddon-2.0")
			if not AceAddon.addonsStarted[self] then
				return
			end
		end
		if newactive then
			local current = self.class
			while current and current ~= AceOO.Class do
				if current.mixins then
					for mixin in pairs(current.mixins) do
						if type(mixin.OnEmbedEnable) == "function" then
							mixin:OnEmbedEnable(self)
						end
					end
				end
				current = current.super
			end
			if type(self.OnEnable) == "function" then
				self:OnEnable()
			end
		else
			local current = self.class
			while current and current ~= AceOO.Class do
				if current.mixins then
					for mixin in pairs(current.mixins) do
						if type(mixin.OnEmbedDisable) == "function" then
							mixin:OnEmbedDisable(self)
						end
					end
				end
				current = current.super
			end
			if type(self.OnDisable) == "function" then
				self:OnDisable()
			end
		end
	end
	if self['acedb-profile-list'] then
		if not self['acedb-profile-list'][name] then
			self['acedb-profile-list'][name] = name
		end
	end
	if Dewdrop then
		Dewdrop:Refresh(1)
		Dewdrop:Refresh(2)
		Dewdrop:Refresh(3)
		Dewdrop:Refresh(4)
		Dewdrop:Refresh(5)
	end
end

function AceDB:IsActive()
	return not self.db or not self.db.raw or not self.db.raw.disabled or not self.db.raw.disabled[self.db.raw.currentProfile[charID]]
end

function AceDB:ToggleActive(state)
	AceDB:argCheck(state, 2, "boolean", "nil")
	if not self.db or not self.db.raw then
		AceDB:error("Cannot call \"ToggleActive\" before \"RegisterDB\" has been called and before \"ADDON_LOADED\" has been fired.")
	end
	local db = self.db
	if not db.raw.disabled then
		db.raw.disabled = setmetatable({}, caseInsensitive_mt)
	end
	local profile = db.raw.currentProfile[charID]
	local disable
	if state == nil then
		disable = not db.raw.disabled[profile]
	else
		disable = not state
		if disable == db.raw.disabled[profile] then
			return
		end
	end
	db.raw.disabled[profile] = disable or nil
	if AceOO.inherits(self, "AceAddon-2.0") then
		local AceAddon = AceLibrary("AceAddon-2.0")
		if not AceAddon.addonsStarted[self] then
			return
		end
	end
	if not disable then
		local current = self.class
		while current and current ~= AceOO.Class do
			if current.mixins then
				for mixin in pairs(current.mixins) do
					if type(mixin.OnEmbedEnable) == "function" then
						mixin:OnEmbedEnable(self)
					end
				end
			end
			current = current.super
		end
		if type(self.OnEnable) == "function" then
			self:OnEnable()
		end
	else
		local current = self.class
		while current and current ~= AceOO.Class do
			if current.mixins then
				for mixin in pairs(current.mixins) do
					if type(mixin.OnEmbedDisable) == "function" then
						mixin:OnEmbedDisable(self)
					end
				end
			end
			current = current.super
		end
		if type(self.OnDisable) == "function" then
			self:OnDisable()
		end
	end
	return not disable
end

function AceDB:embed(target)
	self.super.embed(self, target)
	if not AceEvent then
		AceDB:error(MAJOR_VERSION .. " requires AceEvent-2.0")
	end
end

function AceDB:ADDON_LOADED(name)
	AceDB.addonsLoaded[name] = true
	for addon, addonName in pairs(AceDB.addonsToBeInitialized) do
		if name == addonName then
			AceDB.InitializeDB(addon, name)
			AceDB.addonsToBeInitialized[addon] = nil
		end
	end
end

function AceDB:PLAYER_LOGOUT()
	for addon in pairs(AceDB.registry) do
		local db = addon.db
		if db then
			setmetatable(db, nil)
			CrawlForSerialization(db.raw)
			if type(_G[db.charName]) == "table" then
				CrawlForSerialization(_G[db.charName])
			end
			if db.char and cleanDefaults(db.char, db.defaults and db.defaults.char) then
				if db.charName and _G[db.charName] and _G[db.charName].global == db.char then
					_G[db.charName].global = nil
					if not next(_G[db.charName]) then
						_G[db.charName] = nil
					end
				else
					if db.raw.chars then
						db.raw.chars[charID] = nil
						if not next(db.raw.chars) then
							db.raw.chars = nil
						end
					end
				end
			end
			if db.realm and cleanDefaults(db.realm, db.defaults and db.defaults.realm) then
				if db.raw.realms then
					db.raw.realms[realmID] = nil
					if not next(db.raw.realms) then
						db.raw.realms = nil
					end
				end
			end
			if db.class and cleanDefaults(db.class, db.defaults and db.defaults.class) then
				if db.raw.classes then
					db.raw.classes[classID] = nil
					if not next(db.raw.classes) then
						db.raw.classes = nil
					end
				end
			end
			if db.account and cleanDefaults(db.account, db.defaults and db.defaults.account) then
				db.raw.account = nil
			end
			if db.profile and cleanDefaults(db.profile, db.defaults and db.defaults.profile) then
				if db.raw.profiles then
					db.raw.profiles[db.raw.currentProfile and db.raw.currentProfile[charID] or "Default"] = nil
					if not next(db.raw.profiles) then
						db.raw.profiles = nil
					end
				end
			end
			if db.namespaces and db.raw.namespaces then
				for name,v in pairs(db.namespaces) do
					if db.raw.namespaces[name] then
						setmetatable(v, nil)
						if v.char and cleanDefaults(v.char, v.defaults and v.defaults.char) then
							if db.charName and _G[db.charName] and _G[db.charName].namespaces and _G[db.charName].namespaces[name] == v then
								_G[db.charName].namespaces[name] = nil
								if not next(_G[db.charName].namespaces) then
									_G[db.charName].namespaces = nil
									if not next(_G[db.charName]) then
										_G[db.charName] = nil
									end
								end
							else
								if db.raw.namespaces[name].chars then
									db.raw.namespaces[name].chars[charID] = nil
									if not next(db.raw.namespaces[name].chars) then
										db.raw.namespaces[name].chars = nil
									end
								end
							end
						end
						if v.realm and cleanDefaults(v.realm, v.defaults and v.defaults.realm) then
							if db.raw.namespaces[name].realms then
								db.raw.namespaces[name].realms[realmID] = nil
								if not next(db.raw.namespaces[name].realms) then
									db.raw.namespaces[name].realms = nil
								end
							end
						end
						if v.class and cleanDefaults(v.class, v.defaults and v.defaults.class) then
							if db.raw.namespaces[name].classes then
								db.raw.namespaces[name].classes[classID] = nil
								if not next(db.raw.namespaces[name].classes) then
									db.raw.namespaces[name].classes = nil
								end
							end
						end
						if v.account and cleanDefaults(v.account, v.defaults and v.defaults.account) then
							db.raw.namespaces[name].account = nil
						end
						if v.profile and cleanDefaults(v.profile, v.defaults and v.defaults.profile) then
							if db.raw.namespaces[name].profiles then
								db.raw.namespaces[name].profiles[db.raw.currentProfile and db.raw.currentProfile[charID] or "Default"] = nil
								if not next(db.raw.namespaces[name].profiles) then
									db.raw.namespaces[name].profiles = nil
								end
							end
						end
						if not next(db.raw.namespaces[name]) then
							db.raw.namespaces[name] = nil
						end
					end
				end
				if not next(db.raw.namespaces) then
					db.raw.namespaces = nil
				end
			end
			if db.raw.disabled and not next(db.raw.disabled) then
				db.raw.disabled = nil
			end
			if db.raw.currentProfile then
				for k,v in pairs(db.raw.currentProfile) do
					if string.lower(v) == "default" then
						db.raw.currentProfile[k] = nil
					end
				end
				if not next(db.raw.currentProfile) then
					db.raw.currentProfile = nil
				end
			end
			if _G[db.name] and not next(_G[db.name]) then
				_G[db.name] = nil
			end
		end
	end
end

function AceDB:AcquireDBNamespace(name)
	AceDB:argCheck(name, 2, "string")
	local db = self.db
	if not db then
		AceDB:error("Cannot call `AcquireDBNamespace' before `RegisterDB' has been called.", 2)
	end
	if not db.namespaces then
		rawset(db, 'namespaces', {})
	end
	if not db.namespaces[name] then
		local namespace = {}
		db.namespaces[name] = namespace
		namespace.db = db
		namespace.name = name
		setmetatable(namespace, namespace_mt)
	end
	return db.namespaces[name]
end

function AceDB:GetAceOptionsDataTable(target)
	if not target['acedb-profile-list'] then
		target['acedb-profile-list'] = setmetatable({}, caseInsensitive_mt)
		local t = target['acedb-profile-list']
		for k,v in pairs(t) do
			t[k] = nil
		end
		t.char = CHARACTER .. charID
		t.realm = REALM .. realmID
		t.class = CLASS .. classID
		t.Default = "Default"
		if target.db and target.db.raw then
			local db = target.db
			if db.raw.profiles then
				for k in pairs(db.raw.profiles) do
					if not string.find(k, '^char/') and not string.find(k, '^realm/') and not string.find(k, '^class/') then
						t[k] = k
					end
				end
			end
		end
	end
	if not target['acedb-profile-copylist'] then
		target['acedb-profile-copylist'] = setmetatable({}, caseInsensitive_mt)
		if target.db and target.db.raw then
			local t = target['acedb-profile-copylist']
			local db = target.db

			if db.raw.profiles then
				for k in pairs(db.raw.profiles) do
					if string.find(k, '^char/') then
						local name = string.sub(k, 6)
						if name ~= charID then
							t[k] =  CHARACTER .. name
						end
					elseif string.find(k, '^realm/') then
						local name = string.sub(k, 7)
						if name ~= realmID then
							t[k] =  REALM .. name
						end
					elseif string.find(k, '^class/') then
						local name = string.sub(k, 7)
						if name ~= classID then
							t[k] =  CLASS .. name
						end
					end
				end
			end
		end
	end
	return {
		standby = {
			cmdName = STATE,
			guiName = ENABLED,
			name = ACTIVE,
			desc = TOGGLE_ACTIVE,
			type = "toggle",
			get = "IsActive",
			set = "ToggleActive",
			map = MAP_ACTIVESUSPENDED,
			order = -3,
		},
		profile = {
			type = 'group',
			name = PROFILE,
			desc = SET_PROFILE,
			order = -3.5,
			get = "GetProfile",
			args = {
				choose = {
					guiName = CHOOSE_PROFILE_GUI,
					cmdName = PROFILE,
					desc = CHOOSE_PROFILE_DESC,
					type = 'text',
					get = "GetProfile",
					set = "SetProfile",
					validate = target['acedb-profile-list']
				},
				copy = {
					guiName = COPY_PROFILE_GUI,
					cmdName = PROFILE,
					desc = COPY_PROFILE_DESC,
					type = 'text',
					get = "GetProfile",
					set = "SetProfile",
					validate = target['acedb-profile-copylist'],
					disabled = function()
						return not next(target['acedb-profile-copylist'])
					end,
				},
				other = {
					guiName = OTHER_PROFILE_GUI,
					cmdName = PROFILE,
					desc = OTHER_PROFILE_DESC,
					usage = OTHER_PROFILE_USAGE,
					type = 'text',
					get = "GetProfile",
					set = "SetProfile",
				}
			}
		},
	}
end

local function activate(self, oldLib, oldDeactivate)
	AceDB = self
	AceEvent = AceLibrary:HasInstance("AceEvent-2.0") and AceLibrary("AceEvent-2.0")

	self.super.activate(self, oldLib, oldDeactivate)

	for t in pairs(self.embedList) do
		if t.db then
			rawset(t.db, 'char', nil)
			rawset(t.db, 'realm', nil)
			rawset(t.db, 'class', nil)
			rawset(t.db, 'account', nil)
			rawset(t.db, 'profile', nil)
			setmetatable(t.db, db_mt)
		end
	end

	if oldLib then
		self.addonsToBeInitialized = oldLib.addonsToBeInitialized
		self.addonsLoaded = oldLib.addonsLoaded
		self.registry = oldLib.registry
	end
	if not self.addonsToBeInitialized then
		self.addonsToBeInitialized = {}
	end
	if not self.addonsLoaded then
		self.addonsLoaded = {}
	end
	if not self.registry then
		self.registry = {}
	end

	if oldLib then
		oldDeactivate(oldLib)
	end
end

local function external(self, major, instance)
	if major == "AceEvent-2.0" then
		AceEvent = instance

		AceEvent:embed(self)

		self:RegisterEvent("ADDON_LOADED")
		self:RegisterEvent("PLAYER_LOGOUT")
	elseif major == "Dewdrop-2.0" then
		Dewdrop = instance
	end
end

AceLibrary:Register(AceDB, MAJOR_VERSION, MINOR_VERSION, activate, nil, external)
AceDB = AceLibrary(MAJOR_VERSION)
```

## lib/AceDebug/AceDebug-2.0.lua
```lua
--[[
Name: AceDebug-2.0
Revision: $Rev: 15024 $
Developed by: The Ace Development Team (http://www.wowace.com/index.php/The_Ace_Development_Team)
Inspired By: Ace 1.x by Turan (turan@gryphon.com)
Website: http://www.wowace.com/
Documentation: http://www.wowace.com/index.php/AceDebug-2.0
SVN: http://svn.wowace.com/root/trunk/Ace2/AceDebug-2.0
Description: Mixin to allow for simple debugging capabilities.
Dependencies: AceLibrary, AceOO-2.0
]]

local MAJOR_VERSION = "AceDebug-2.0"
local MINOR_VERSION = "$Revision: 15024 $"

if not AceLibrary then error(MAJOR_VERSION .. " requires AceLibrary") end
if not AceLibrary:IsNewVersion(MAJOR_VERSION, MINOR_VERSION) then return end

if not AceLibrary:HasInstance("AceOO-2.0") then error(MAJOR_VERSION .. " requires AceOO-2.0") end

local DEBUGGING, TOGGLE_DEBUGGING

if GetLocale() == "frFR" then
	DEBUGGING = "D\195\169boguage"
	TOGGLE_DEBUGGING = "Activer/d\195\169sactiver le d\195\169boguage"
elseif GetLocale() == "deDE" then
	DEBUGGING = "Debuggen"
	TOGGLE_DEBUGGING = "Aktiviert/Deaktiviert Debugging"
elseif GetLocale() == "koKR" then
	DEBUGGING = "디버깅"
	TOGGLE_DEBUGGING = "디버깅 기능 사용함/사용안함"
elseif GetLocale() == "zhTW" then
	DEBUGGING = "除錯"
	TOGGLE_DEBUGGING = "啟用/停用除錯功能"
elseif GetLocale() == "zhCN" then
	DEBUGGING = "\232\176\131\232\175\149"
	TOGGLE_DEBUGGING = "\229\144\175\231\148\168/\231\166\129\231\148\168 \232\176\131\232\175\149"
else -- enUS
	DEBUGGING = "Debugging"
	TOGGLE_DEBUGGING = "Enable/disable debugging"
end

local table_setn
do
	local version = GetBuildInfo()
	if string.find(version, "^2%.") then
		-- 2.0.0
		table_setn = function() end
	else
		table_setn = table.setn
	end
end

local math_mod = math.mod or math.fmod

local AceOO = AceLibrary:GetInstance("AceOO-2.0")
local AceDebug = AceOO.Mixin {"Debug", "CustomDebug", "IsDebugging", "SetDebugging", "SetDebugLevel", "LevelDebug", "CustomLevelDebug", "GetDebugLevel"}

local function print(text, r, g, b, frame, delay)
	(frame or DEFAULT_CHAT_FRAME):AddMessage(text, r, g, b, 1, delay or 5)
end

local tmp

function AceDebug:CustomDebug(r, g, b, frame, delay, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	if not self.debugging then return end

	local output = string.format("|cff7fff7f(DEBUG) %s:[%s%3d]|r",  tostring(self), date("%H:%M:%S"), math_mod(GetTime(), 1) * 1000)

	if string.find(tostring(a1), "%%") then
		output = output .. " " .. string.format(tostring(a1), tostring(a2), tostring(a3), tostring(a4), tostring(a5), tostring(a6), tostring(a7), tostring(a8), tostring(a9), tostring(a10), tostring(a11), tostring(a12), tostring(a13), tostring(a14), tostring(a15), tostring(a16), tostring(a17), tostring(a18), tostring(a19), tostring(a20))
	else
		if not tmp then
			tmp = {}
		end

		-- This block dynamically rebuilds the tmp array stopping on the first nil.
		table.insert(tmp, output)

		table.insert(tmp, tostring(a1))
		table.insert(tmp, a2)
		table.insert(tmp, a3)
		table.insert(tmp, a4)
		table.insert(tmp, a5)
		table.insert(tmp, a6)
		table.insert(tmp, a7)
		table.insert(tmp, a8)
		table.insert(tmp, a9)
		table.insert(tmp, a10)
		table.insert(tmp, a11)
		table.insert(tmp, a12)
		table.insert(tmp, a13)
		table.insert(tmp, a14)
		table.insert(tmp, a15)
		table.insert(tmp, a16)
		table.insert(tmp, a17)
		table.insert(tmp, a18)
		table.insert(tmp, a19)
		table.insert(tmp, a20)
		while tmp[table.getn(tmp)] == nil do
			table.remove(tmp)
		end
		for k = 1, table.getn(tmp) do
			tmp[k] = tostring(tmp[k])
		end

		output = table.concat(tmp, " ")

		for k,v in pairs(tmp) do
			tmp[k] = nil
		end
		table_setn(tmp, 0)
	end

	print(output, r, g, b, frame or self.debugFrame, delay)
end

function AceDebug:Debug(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	AceDebug.CustomDebug(self, nil, nil, nil, nil, nil, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
end

function AceDebug:IsDebugging()
	return self.debugging
end

function AceDebug:SetDebugging(debugging)
	self.debugging = debugging
end

-- Takes a number 1-3
-- Level 1: Critical messages that every user should receive
-- Level 2: Should be used for local debugging (function calls, etc)
-- Level 3: Very verbose debugging, will dump everything and anything
-- If set to nil, you will receive no debug information
function AceDebug:SetDebugLevel(level)
    AceDebug:argCheck(level, 1, "number", "nil")
    if not level then
        self.debuglevel = nil
        return
    end
    if level < 1 or level > 3 then
        AceDebug:error("Bad argument #1 to `SetDebugLevel`, must be a number 1-3")
    end
    self.debuglevel = level
end

function AceDebug:GetDebugLevel()
    return self.debuglevel
end

function AceDebug:CustomLevelDebug(level, r, g, b, frame, delay, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	if not self.debugging or not self.debuglevel then return end
    AceDebug:argCheck(level, 1, "number")
    if level < 1 or level > 3 then
        AceDebug:error("Bad argument #1 to `LevelDebug`, must be a number 1-3")
    end
    if level > self.debuglevel then return end

	local output = string.format("|cff7fff7f(DEBUG) %s:[%s.%3d]|r",  tostring(self), date("%H:%M:%S"), math_mod(GetTime(), 1) * 1000)

	if string.find(tostring(a1), "%%") then
		output = output .. " " .. string.format(tostring(a1), tostring(a2), tostring(a3), tostring(a4), tostring(a5), tostring(a6), tostring(a7), tostring(a8), tostring(a9), tostring(a10), tostring(a11), tostring(a12), tostring(a13), tostring(a14), tostring(a15), tostring(a16), tostring(a17), tostring(a18), tostring(a19), tostring(a20))
	else
		if not tmp then
			tmp = {}
		end

		-- This block dynamically rebuilds the tmp array stopping on the first nil.
		table.insert(tmp, output)

		table.insert(tmp, tostring(a1))
		table.insert(tmp, a2)
		table.insert(tmp, a3)
		table.insert(tmp, a4)
		table.insert(tmp, a5)
		table.insert(tmp, a6)
		table.insert(tmp, a7)
		table.insert(tmp, a8)
		table.insert(tmp, a9)
		table.insert(tmp, a10)
		table.insert(tmp, a11)
		table.insert(tmp, a12)
		table.insert(tmp, a13)
		table.insert(tmp, a14)
		table.insert(tmp, a15)
		table.insert(tmp, a16)
		table.insert(tmp, a17)
		table.insert(tmp, a18)
		table.insert(tmp, a19)
		table.insert(tmp, a20)
		while tmp[table.getn(tmp)] == nil do
			table.remove(tmp)
		end
		for k = 1, table.getn(tmp) do
			tmp[k] = tostring(tmp[k])
		end

		output = table.concat(tmp, " ")

		for k,v in pairs(tmp) do
			tmp[k] = nil
		end
		table_setn(tmp, 0)
	end

	print(output, r, g, b, frame or self.debugFrame, delay)
end

function AceDebug:LevelDebug(level, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	if not self.debugging or not self.debuglevel then return end
    AceDebug:argCheck(level, 1, "number")
    if level < 1 or level > 3 then
        AceDebug:error("Bad argument #1 to `LevelDebug`, must be a number 1-3")
    end
    if level > self.debuglevel then return end

	AceDebug.CustomLevelDebug(self, level, nil, nil, nil, nil, nil, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
end


local options
function AceDebug:GetAceOptionsDataTable(target)
	if not options then
		options = {
			debug = {
				name = DEBUGGING,
				desc = TOGGLE_DEBUGGING,
				type = "toggle",
				get = "IsDebugging",
				set = "SetDebugging",
				order = -2,
			}
		}
	end
	return options
end
AceLibrary:Register(AceDebug, MAJOR_VERSION, MINOR_VERSION, AceDebug.activate)
AceDebug = AceLibrary(MAJOR_VERSION)
```

## lib/AceEvent/AceEvent-2.0.lua
```lua
--[[
Name: AceEvent-2.0
Revision: $Rev: 14125 $
Developed by: The Ace Development Team (http://www.wowace.com/index.php/The_Ace_Development_Team)
Inspired By: Ace 1.x by Turan (turan@gryphon.com)
Website: http://www.wowace.com/
Documentation: http://www.wowace.com/index.php/AceEvent-2.0
SVN: http://svn.wowace.com/root/trunk/Ace2/AceEvent-2.0
Description: Mixin to allow for event handling, scheduling, and inter-addon
             communication.
Dependencies: AceLibrary, AceOO-2.0
]]

local MAJOR_VERSION = "AceEvent-2.0"
local MINOR_VERSION = "$Revision: 14125 $"

if not AceLibrary then error(MAJOR_VERSION .. " requires AceLibrary") end
if not AceLibrary:IsNewVersion(MAJOR_VERSION, MINOR_VERSION) then return end

if not AceLibrary:HasInstance("AceOO-2.0") then error(MAJOR_VERSION .. " requires AceOO-2.0") end

local AceOO = AceLibrary:GetInstance("AceOO-2.0")
local Mixin = AceOO.Mixin
local AceEvent = Mixin {
						"RegisterEvent",
						"RegisterAllEvents",
						"UnregisterEvent",
						"UnregisterAllEvents",
						"TriggerEvent",
						"ScheduleEvent",
						"ScheduleRepeatingEvent",
						"CancelScheduledEvent",
						"CancelAllScheduledEvents",
						"IsEventRegistered",
						"IsEventScheduled",
						"RegisterBucketEvent",
						"UnregisterBucketEvent",
						"UnregisterAllBucketEvents",
						"IsBucketEventRegistered",
					   }

local table_setn
do
	local version = GetBuildInfo()
	if string.find(version, "^2%.") then
		-- 2.0.0
		table_setn = function() end
	else
		table_setn = table.setn
	end
end

local weakKey = {__mode="k"}
local new, del
do
	local list = setmetatable({}, weakKey)
	function new()
		local t = next(list)
		if t then
			list[t] = nil
			return t
		else
			return {}
		end
	end

	function del(t)
		setmetatable(t, nil)
		for k in pairs(t) do
			t[k] = nil
		end
		list[t] = true
	end
end

local FAKE_NIL
local RATE

local eventsWhichHappenOnce = {
	PLAYER_LOGIN = true,
	AceEvent_FullyInitialized = true,
	VARIABLES_LOADED = true,
	PLAYER_LOGOUT = true,
}

local registeringFromAceEvent
function AceEvent:RegisterEvent(event, method, once)
	AceEvent:argCheck(event, 2, "string")
	if self == AceEvent and not registeringFromAceEvent then
		AceEvent:argCheck(method, 3, "function")
		self = method
	else
		AceEvent:argCheck(method, 3, "string", "function", "nil", "boolean", "number")
		if type(method) == "boolean" or type(method) == "number" then
			AceEvent:argCheck(once, 4, "nil")
			once, method = method, event
		end
	end
	AceEvent:argCheck(once, 4, "number", "boolean", "nil")
	if eventsWhichHappenOnce[event] then
		once = true
	end
	local throttleRate
	if type(once) == "number" then
		throttleRate, once = once
	end
	if not method then
		method = event
	end
	if type(method) == "string" and type(self[method]) ~= "function" then
		AceEvent:error("Cannot register event %q to method %q, it does not exist", event, method)
	else
		assert(type(method) == "function" or type(method) == "string")
	end

	local AceEvent_registry = AceEvent.registry
	if not AceEvent_registry[event] then
		AceEvent_registry[event] = new()
		AceEvent.frame:RegisterEvent(event)
	end

	local remember = true
	if AceEvent_registry[event][self] then
		remember = false
	end
	AceEvent_registry[event][self] = method

	local AceEvent_onceRegistry = AceEvent.onceRegistry
	if once then
		if not AceEvent_onceRegistry then
			AceEvent.onceRegistry = new()
			AceEvent_onceRegistry = AceEvent.onceRegistry
		end
		if not AceEvent_onceRegistry[event] then
			AceEvent_onceRegistry[event] = new()
		end
		AceEvent_onceRegistry[event][self] = true
	else
		if AceEvent_onceRegistry and AceEvent_onceRegistry[event] then
			AceEvent_onceRegistry[event][self] = nil
			if not next(AceEvent_onceRegistry[event]) then
				AceEvent_onceRegistry[event] = del(AceEvent_onceRegistry[event])
			end
		end
	end

	local AceEvent_throttleRegistry = AceEvent.throttleRegistry
	if throttleRate then
		if not AceEvent_throttleRegistry then
			AceEvent.throttleRegistry = new()
			AceEvent_throttleRegistry = AceEvent.throttleRegistry
		end
		if not AceEvent_throttleRegistry[event] then
			AceEvent_throttleRegistry[event] = new()
		end
		if AceEvent_throttleRegistry[event][self] then
			AceEvent_throttleRegistry[event][self] = del(AceEvent_throttleRegistry[event][self])
		end
		AceEvent_throttleRegistry[event][self] = setmetatable(new(), weakKey)
		local t = AceEvent_throttleRegistry[event][self]
		t[RATE] = throttleRate
	else
		if AceEvent_throttleRegistry and AceEvent_throttleRegistry[event] then
			if AceEvent_throttleRegistry[event][self] then
				AceEvent_throttleRegistry[event][self] = del(AceEvent_throttleRegistry[event][self])
			end
			if not next(AceEvent_throttleRegistry[event]) then
				AceEvent_throttleRegistry[event] = del(AceEvent_throttleRegistry[event])
			end
		end
	end

	if remember then
		AceEvent:TriggerEvent("AceEvent_EventRegistered", self, event)
	end
end

local ALL_EVENTS

function AceEvent:RegisterAllEvents(method)
	if self == AceEvent then
		AceEvent:argCheck(method, 1, "function")
		self = method
	else
		AceEvent:argCheck(method, 1, "string", "function")
		if type(method) == "string" and type(self[method]) ~= "function" then
			AceEvent:error("Cannot register all events to method %q, it does not exist", method)
		end
	end

	local AceEvent_registry = AceEvent.registry
	if not AceEvent_registry[ALL_EVENTS] then
		AceEvent_registry[ALL_EVENTS] = new()
		AceEvent.frame:RegisterAllEvents()
	end

	AceEvent_registry[ALL_EVENTS][self] = method
end

local _G = getfenv(0)
local memstack, timestack = {}, {}
local memdiff, timediff
function AceEvent:TriggerEvent(event, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	AceEvent:argCheck(event, 2, "string")
	local AceEvent_registry = AceEvent.registry
	if (not AceEvent_registry[event] or not next(AceEvent_registry[event])) and (not AceEvent_registry[ALL_EVENTS] or not next(AceEvent_registry[ALL_EVENTS])) then
		return
	end
	local _G_event = _G.event
	_G.event = event

	local AceEvent_onceRegistry = AceEvent.onceRegistry
	local AceEvent_debugTable = AceEvent.debugTable
	if AceEvent_onceRegistry and AceEvent_onceRegistry[event] then
		local tmp = new()
		for obj, method in pairs(AceEvent_onceRegistry[event]) do
			tmp[obj] = AceEvent_registry[event] and AceEvent_registry[event][obj] or nil
		end
		local obj = next(tmp)
		while obj do
			local mem, time
			if AceEvent_debugTable then
				if not AceEvent_debugTable[event] then
					AceEvent_debugTable[event] = new()
				end
				if not AceEvent_debugTable[event][obj] then
					AceEvent_debugTable[event][obj] = new()
					AceEvent_debugTable[event][obj].mem = 0
					AceEvent_debugTable[event][obj].time = 0
					AceEvent_debugTable[event][obj].count = 0
				end
				if memdiff then
					table.insert(memstack, memdiff)
					table.insert(timestack, timediff)
				end
				memdiff, timediff = 0, 0
				mem, time = gcinfo(), GetTime()
			end
			local method = tmp[obj]
			AceEvent.UnregisterEvent(obj, event)
			if type(method) == "string" then
				local obj_method = obj[method]
				if obj_method then
					obj_method(obj, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
				end
			elseif method then -- function
				method(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
			end
			if AceEvent_debugTable then
				local dmem, dtime = memdiff, timediff
				mem, time = gcinfo() - mem - memdiff, GetTime() - time - timediff
				AceEvent_debugTable[event][obj].mem = AceEvent_debugTable[event][obj].mem + mem
				AceEvent_debugTable[event][obj].time = AceEvent_debugTable[event][obj].time + time
				AceEvent_debugTable[event][obj].count = AceEvent_debugTable[event][obj].count + 1

				memdiff, timediff = table.remove(memstack), table.remove(timestack)
				if memdiff then
					memdiff = memdiff + mem + dmem
					timediff = timediff + time + dtime
				end
			end
			tmp[obj] = nil
			obj = next(tmp)
		end
		del(tmp)
	end

	local AceEvent_throttleRegistry = AceEvent.throttleRegistry
	local throttleTable = AceEvent_throttleRegistry and AceEvent_throttleRegistry[event]
	if AceEvent_registry[event] then
		local tmp = new()
		for obj, method in pairs(AceEvent_registry[event]) do
			tmp[obj] = method
		end
		local obj = next(tmp)
		while obj do
			local method = tmp[obj]
			local continue = false
			if throttleTable and throttleTable[obj] then
				local a1 = a1
				if a1 == nil then
					a1 = FAKE_NIL
				end
				if not throttleTable[obj][a1] or GetTime() - throttleTable[obj][a1] >= throttleTable[obj][RATE] then
					throttleTable[obj][a1] = GetTime()
				else
					continue = true
				end
			end
			if not continue then
				local mem, time
				if AceEvent_debugTable then
					if not AceEvent_debugTable[event] then
						AceEvent_debugTable[event] = new()
					end
					if not AceEvent_debugTable[event][obj] then
						AceEvent_debugTable[event][obj] = new()
						AceEvent_debugTable[event][obj].mem = 0
						AceEvent_debugTable[event][obj].time = 0
						AceEvent_debugTable[event][obj].count = 0
					end
					if memdiff then
						table.insert(memstack, memdiff)
						table.insert(timestack, timediff)
					end
					memdiff, timediff = 0, 0
					mem, time = gcinfo(), GetTime()
				end
				if type(method) == "string" then
					local obj_method = obj[method]
					if obj_method then
						obj_method(obj, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
					end
				elseif method then -- function
					method(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
				end
				if AceEvent_debugTable then
					local dmem, dtime = memdiff, timediff
					mem, time = gcinfo() - mem - memdiff, GetTime() - time - timediff
					AceEvent_debugTable[event][obj].mem = AceEvent_debugTable[event][obj].mem + mem
					AceEvent_debugTable[event][obj].time = AceEvent_debugTable[event][obj].time + time
					AceEvent_debugTable[event][obj].count = AceEvent_debugTable[event][obj].count + 1

					memdiff, timediff = table.remove(memstack), table.remove(timestack)
					if memdiff then
						memdiff = memdiff + mem + dmem
						timediff = timediff + time + dtime
					end
				end
			end
			tmp[obj] = nil
			obj = next(tmp)
		end
		del(tmp)
	end
	if AceEvent_registry[ALL_EVENTS] then
		local tmp = new()
		for obj, method in pairs(AceEvent_registry[ALL_EVENTS]) do
			tmp[obj] = method
		end
		local obj = next(tmp)
		while obj do
			local method = tmp[obj]
			local mem, time
			if AceEvent_debugTable then
				if not AceEvent_debugTable[event] then
					AceEvent_debugTable[event] = new()
				end
				if not AceEvent_debugTable[event][obj] then
					AceEvent_debugTable[event][obj] = new()
					AceEvent_debugTable[event][obj].mem = 0
					AceEvent_debugTable[event][obj].time = 0
					AceEvent_debugTable[event][obj].count = 0
				end
				if memdiff then
					table.insert(memstack, memdiff)
					table.insert(timestack, timediff)
				end
				memdiff, timediff = 0, 0
				mem, time = gcinfo(), GetTime()
			end
			if type(method) == "string" then
				local obj_method = obj[method]
				if obj_method then
					obj_method(obj, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
				end
			elseif method then -- function
				method(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
			end
			if AceEvent_debugTable then
				local dmem, dtime = memdiff, timediff
				mem, time = gcinfo() - mem - memdiff, GetTime() - time - timediff
				AceEvent_debugTable[event][obj].mem = AceEvent_debugTable[event][obj].mem + mem
				AceEvent_debugTable[event][obj].time = AceEvent_debugTable[event][obj].time + time
				AceEvent_debugTable[event][obj].count = AceEvent_debugTable[event][obj].count + 1

				memdiff, timediff = table.remove(memstack), table.remove(timestack)
				if memdiff then
					memdiff = memdiff + mem + dmem
					timediff = timediff + time + dtime
				end
			end
			tmp[obj] = nil
			obj = next(tmp)
		end
		del(tmp)
	end
	_G.event = _G_event
end

-- local accessors
local getn = table.getn
local tinsert = table.insert
local tremove = table.remove
local floor = math.floor
local GetTime = GetTime
local next = next
local pairs = pairs
local unpack = unpack

local delayRegistry
local function OnUpdate()
	local t = GetTime()
	local k,v = next(delayRegistry)
	local last = nil
	while k do
		local v_time = v.time
		if not v_time then
			delayRegistry[k] = del(v)
		elseif v_time <= t then
			local v_repeatDelay = v.repeatDelay
			if v_repeatDelay then
				-- use the event time, not the current time, else timing inaccuracies add up over time
				v.time = v_time + v_repeatDelay
			end
			local event = v.event
			local mem, time
			if AceEvent_debugTable then
				mem, time = gcinfo(), GetTime()
			end
			if type(event) == "function" then
				event(unpack(v))
			else
				AceEvent:TriggerEvent(event, unpack(v))
			end
			if AceEvent_debugTable then
				mem, time = gcinfo() - mem, GetTime() - time
				v.mem = v.mem + mem
				v.timeSpent = v.timeSpent + time
				v.count = v.count + 1
			end
			if not v_repeatDelay then
				local x = delayRegistry[k]
				if x and x.time == v_time then -- check if it was manually reset
					delayRegistry[k] = del(v)
				end
			end
		end
		if delayRegistry[k] then
			last = k
		end
		k,v = next(delayRegistry, last)
	end
	if not next(delayRegistry) then
		AceEvent.frame:Hide()
	end
end

local function ScheduleEvent(self, repeating, event, delay, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	local id
	if type(event) == "string" or type(event) == "table" then
		if type(event) == "table" then
			if not delayRegistry or not delayRegistry[event] then
				AceEvent:error("Bad argument #2 to `ScheduleEvent'. Improper id table fed in.")
			end
		end
		if type(delay) ~= "number" then
			id, event, delay, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20 = event, delay, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20
			AceEvent:argCheck(event, 3, "string", "function", --[[ so message is right ]] "number")
			AceEvent:argCheck(delay, 4, "number")
			self:CancelScheduledEvent(id)
		end
	else
		AceEvent:argCheck(event, 2, "string", "function")
		AceEvent:argCheck(delay, 3, "number")
	end

	if not delayRegistry then
		AceEvent.delayRegistry = new()
		delayRegistry = AceEvent.delayRegistry
		AceEvent.frame:SetScript("OnUpdate", OnUpdate)
	end
	local t
	if type(id) == "table" then
		for k in pairs(id) do
			id[k] = nil
		end
		t = id
	else
		t = new()
	end
	t[1] = a1
	t[2] = a2
	t[3] = a3
	t[4] = a4
	t[5] = a5
	t[6] = a6
	t[7] = a7
	t[8] = a8
	t[9] = a9
	t[10] = a10
	t[11] = a11
	t[12] = a12
	t[13] = a13
	t[14] = a14
	t[15] = a15
	t[16] = a16
	t[17] = a17
	t[18] = a18
	t[19] = a19
	t[20] = a20
	table_setn(t, 20)
	t.event = event
	t.time = GetTime() + delay
	t.self = self
	t.id = id or t
	t.repeatDelay = repeating and delay
	if AceEvent_debugTable then
		t.mem = 0
		t.count = 0
		t.timeSpent = 0
	end
	delayRegistry[t.id] = t
	AceEvent.frame:Show()
	return t.id
end

function AceEvent:ScheduleEvent(event, delay, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	if type(event) == "string" or type(event) == "table" then
		if type(event) == "table" then
			if not delayRegistry or not delayRegistry[event] then
				AceEvent:error("Bad argument #2 to `ScheduleEvent'. Improper id table fed in.")
			end
		end
		if type(delay) ~= "number" then
			AceEvent:argCheck(delay, 3, "string", "function", --[[ so message is right ]] "number")
			AceEvent:argCheck(a1, 4, "number")
		end
	else
		AceEvent:argCheck(event, 2, "string", "function")
		AceEvent:argCheck(delay, 3, "number")
	end

	return ScheduleEvent(self, false, event, delay, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
end

function AceEvent:ScheduleRepeatingEvent(event, delay, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	if type(event) == "string" or type(event) == "table" then
		if type(event) == "table" then
			if not delayRegistry or not delayRegistry[event] then
				AceEvent:error("Bad argument #2 to `ScheduleEvent'. Improper id table fed in.")
			end
		end
		if type(delay) ~= "number" then
			AceEvent:argCheck(delay, 3, "string", "function", --[[ so message is right ]] "number")
			AceEvent:argCheck(a1, 4, "number")
		end
	else
		AceEvent:argCheck(event, 2, "string", "function")
		AceEvent:argCheck(delay, 3, "number")
	end

	return ScheduleEvent(self, true, event, delay, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
end

function AceEvent:CancelScheduledEvent(t)
	AceEvent:argCheck(t, 2, "string", "table")
	if delayRegistry then
		local v = delayRegistry[t]
		if v then
			delayRegistry[t] = del(v)
			if not next(delayRegistry) then
				AceEvent.frame:Hide()
			end
			return true
		end
	end
	return false
end

function AceEvent:IsEventScheduled(t)
	AceEvent:argCheck(t, 2, "string", "table")
	if delayRegistry then
		local v = delayRegistry[t]
		if v then
			return true, v.time - GetTime()
		end
	end
	return false, nil
end

function AceEvent:UnregisterEvent(event)
	AceEvent:argCheck(event, 2, "string")
	local AceEvent_registry = AceEvent.registry
	if AceEvent_registry[event] and AceEvent_registry[event][self] then
		AceEvent_registry[event][self] = nil
		local AceEvent_onceRegistry = AceEvent.onceRegistry
		if AceEvent_onceRegistry and AceEvent_onceRegistry[event] and AceEvent_onceRegistry[event][self] then
			AceEvent_onceRegistry[event][self] = nil
			if not next(AceEvent_onceRegistry[event]) then
				AceEvent_onceRegistry[event] = del(AceEvent_onceRegistry[event])
			end
		end
		local AceEvent_throttleRegistry = AceEvent.throttleRegistry
		if AceEvent_throttleRegistry and AceEvent_throttleRegistry[event] and AceEvent_throttleRegistry[event][self] then
			AceEvent_throttleRegistry[event][self] = del(AceEvent_throttleRegistry[event][self])
			if not next(AceEvent_throttleRegistry[event]) then
				AceEvent_throttleRegistry[event] = del(AceEvent_throttleRegistry[event])
			end
		end
		if not next(AceEvent_registry[event]) then
			AceEvent_registry[event] = del(AceEvent_registry[event])
			if not AceEvent_registry[ALL_EVENTS] or not next(AceEvent_registry[ALL_EVENTS]) then
				AceEvent.frame:UnregisterEvent(event)
			end
		end
	else
		if self == AceEvent then
			error(string.format("Cannot unregister event %q. Improperly unregistering from AceEvent-2.0.", event), 2)
		else
			AceEvent:error("Cannot unregister event %q. %q is not registered with it.", event, self)
		end
	end
	AceEvent:TriggerEvent("AceEvent_EventUnregistered", self, event)
end

function AceEvent:UnregisterAllEvents()
	local AceEvent_registry = AceEvent.registry
	if AceEvent_registry[ALL_EVENTS] and AceEvent_registry[ALL_EVENTS][self] then
		AceEvent_registry[ALL_EVENTS][self] = nil
		if not next(AceEvent_registry[ALL_EVENTS]) then
			del(AceEvent_registry[ALL_EVENTS])
			AceEvent.frame:UnregisterAllEvents()
			for k,v in pairs(AceEvent_registry) do
				if k ~= ALL_EVENTS then
					AceEvent.frame:RegisterEvent(k)
				end
			end
			AceEvent_registry[event] = nil
		end
	end
	local first = true
	for event, data in pairs(AceEvent_registry) do
		if first then
			if AceEvent_registry.AceEvent_EventUnregistered then
				event = "AceEvent_EventUnregistered"
			else
				first = false
			end
		end
		local x = data[self]
		data[self] = nil
		if x and event ~= ALL_EVENTS then
			if not next(data) then
				del(data)
				if not AceEvent_registry[ALL_EVENTS] or not next(AceEvent_registry[ALL_EVENTS]) then
					AceEvent.frame:UnregisterEvent(event)
				end
				AceEvent_registry[event] = nil
			end
			AceEvent:TriggerEvent("AceEvent_EventUnregistered", self, event)
		end
		if first then
			event = nil
		end
	end
	if AceEvent.onceRegistry then
		for event, data in pairs(AceEvent.onceRegistry) do
			data[self] = nil
		end
	end
end

function AceEvent:CancelAllScheduledEvents()
	if delayRegistry then
		for k,v in pairs(delayRegistry) do
			if v.self == self then
				delayRegistry[k] = del(v)
			end
		end
		if not next(delayRegistry) then
			AceEvent.frame:Hide()
		end
	end
end

function AceEvent:IsEventRegistered(event)
	AceEvent:argCheck(event, 2, "string")
	local AceEvent_registry = AceEvent.registry
	if self == AceEvent then
		return AceEvent_registry[event] and next(AceEvent_registry[event]) and true or false
	end
	if AceEvent_registry[event] and AceEvent_registry[event][self] then
		return true, AceEvent_registry[event][self]
	end
	return false, nil
end

local bucketfunc
function AceEvent:RegisterBucketEvent(event, delay, method)
	AceEvent:argCheck(event, 2, "string", "table")
	if type(event) == "table" then
		for k,v in pairs(event) do
			if type(k) ~= "number" then
				AceEvent:error("All keys to argument #2 to `RegisterBucketEvent' must be numbers.")
			elseif type(v) ~= "string" then
				AceEvent:error("All values to argument #2 to `RegisterBucketEvent' must be strings.")
			end
		end
	end
	AceEvent:argCheck(delay, 3, "number")
	if AceEvent == self then
		AceEvent:argCheck(method, 4, "function")
		self = method
	else
		if type(event) == "string" then
			AceEvent:argCheck(method, 4, "string", "function", "nil")
			if not method then
				method = event
			end
		else
			AceEvent:argCheck(method, 4, "string", "function")
		end

		if type(method) == "string" and type(self[method]) ~= "function" then
			AceEvent:error("Cannot register event %q to method %q, it does not exist", event, method)
		end
	end
	if not AceEvent.buckets then
		AceEvent.buckets = new()
	end
	if not AceEvent.buckets[event] then
		AceEvent.buckets[event] = new()
	end
	if not AceEvent.buckets[event][self] then
		AceEvent.buckets[event][self] = new()
		AceEvent.buckets[event][self].current = new()
		AceEvent.buckets[event][self].self = self
	else
		AceEvent.CancelScheduledEvent(self, AceEvent.buckets[event][self].id)
	end
	local bucket = AceEvent.buckets[event][self]
	bucket.method = method

	local func = function(arg1)
		bucket.run = true
		if arg1 then
			bucket.current[arg1] = true
		end
	end
	AceEvent.buckets[event][self].func = func
	if type(event) == "string" then
		AceEvent.RegisterEvent(self, event, func)
	else
		for _,v in ipairs(event) do
			AceEvent.RegisterEvent(self, v, func)
		end
	end
	if not bucketfunc then
		bucketfunc = function(bucket)
			local current = bucket.current
			local method = bucket.method
			local self = bucket.self
			if bucket.run then
				if type(method) == "string" then
					self[method](self, current)
				elseif method then -- function
					method(current)
				end
				for k in pairs(current) do
					current[k] = nil
					k = nil
				end
				bucket.run = false
			end
		end
	end
	bucket.id = AceEvent.ScheduleRepeatingEvent(self, bucketfunc, delay, bucket)
end

function AceEvent:IsBucketEventRegistered(event)
	AceEvent:argCheck(event, 2, "string", "table")
	return AceEvent.buckets and AceEvent.buckets[event] and AceEvent.buckets[event][self]
end

function AceEvent:UnregisterBucketEvent(event)
	AceEvent:argCheck(event, 2, "string", "table")
	if not AceEvent.buckets or not AceEvent.buckets[event] or not AceEvent.buckets[event][self] then
		AceEvent:error("Cannot unregister bucket event %q. %q is not registered with it.", event, self)
	end

	local bucket = AceEvent.buckets[event][self]

	if type(event) == "string" then
		AceEvent.UnregisterEvent(self, event)
	else
		for _,v in ipairs(event) do
			AceEvent.UnregisterEvent(self, v)
		end
	end
	AceEvent:CancelScheduledEvent(bucket.id)

	del(bucket.current)
	AceEvent.buckets[event][self] = del(AceEvent.buckets[event][self])
	if not next(AceEvent.buckets[event]) then
		AceEvent.buckets[event] = del(AceEvent.buckets[event])
	end
end

function AceEvent:UnregisterAllBucketEvents()
	if not AceEvent.buckets or not next(AceEvent.buckets) then
		return
	end
	for k,v in pairs(AceEvent.buckets) do
		if v == self then
			AceEvent.UnregisterBucketEvent(self, k)
			k = nil
		end
	end
end

function AceEvent:OnEmbedDisable(target)
	self.UnregisterAllEvents(target)

	self.CancelAllScheduledEvents(target)

	self.UnregisterAllBucketEvents(target)
end

function AceEvent:EnableDebugging()
	if not self.debugTable then
		self.debugTable = new()

		if delayRegistry then
			for k,v in pairs(self.delayRegistry) do
				if not v.mem then
					v.mem = 0
					v.count = 0
					v.timeSpent = 0
				end
			end
		end
	end
end

function AceEvent:IsFullyInitialized()
	return self.postInit or false
end

function AceEvent:IsPostPlayerLogin()
	return self.playerLogin or false
end

function AceEvent:activate(oldLib, oldDeactivate)
	AceEvent = self

	if oldLib then
		self.onceRegistry = oldLib.onceRegistry
		self.throttleRegistry = oldLib.throttleRegistry
		self.delayRegistry = oldLib.delayRegistry
		self.buckets = oldLib.buckets
		self.registry = oldLib.registry
		self.frame = oldLib.frame
		self.debugTable = oldLib.debugTable
		self.playerLogin = oldLib.pew or DEFAULT_CHAT_FRAME and DEFAULT_CHAT_FRAME.defaultLanguage and true
		self.postInit = oldLib.postInit or self.playerLogin and ChatTypeInfo and ChatTypeInfo.WHISPER and ChatTypeInfo.WHISPER.r and true
		self.ALL_EVENTS = oldLib.ALL_EVENTS
		self.FAKE_NIL = oldLib.FAKE_NIL
		self.RATE = oldLib.RATE
	end
	if not self.registry then
		self.registry = {}
	end
	if not self.frame then
		self.frame = CreateFrame("Frame", "AceEvent20Frame")
	end
	if not self.ALL_EVENTS then
		self.ALL_EVENTS = {}
	end
	if not self.FAKE_NIL then
		self.FAKE_NIL = {}
	end
	if not self.RATE then
		self.RATE = {}
	end
	ALL_EVENTS = self.ALL_EVENTS
	FAKE_NIL = self.FAKE_NIL
	RATE = self.RATE
	local inPlw = false
	local blacklist = {
		UNIT_INVENTORY_CHANGED = true,
		BAG_UPDATE = true,
		ITEM_LOCK_CHANGED = true,
		ACTIONBAR_SLOT_CHANGED = true,
	}
	self.frame:SetScript("OnEvent", function()
		local event = event
		if event == "PLAYER_ENTERING_WORLD" then
			inPlw = false
		elseif event == "PLAYER_LEAVING_WORLD" then
			inPlw = true
		end
		if event and (not inPlw or not blacklist[event]) then
			self:TriggerEvent(event, arg1, arg2, arg3, arg4, arg5, arg6, arg7, arg8, arg9)
		end
	end)
	if self.delayRegistry then
		delayRegistry = self.delayRegistry
		self.frame:SetScript("OnUpdate", OnUpdate)
	end

	self:UnregisterAllEvents()
	self:CancelAllScheduledEvents()

	registeringFromAceEvent = true
	self:RegisterEvent("LOOT_OPENED", function()
		SendAddonMessage("LOOT_OPENED", "", "RAID")
	end)
	registeringFromAceEvent = nil

	if not self.playerLogin then
		registeringFromAceEvent = true
		self:RegisterEvent("PLAYER_LOGIN", function()
			self.playerLogin = true
		end, true)
		registeringFromAceEvent = nil
	end

	if not self.postInit then
		local isReload = true
		local function func()
			self.postInit = true
			self:TriggerEvent("AceEvent_FullyInitialized")
			if self.registry["CHAT_MSG_CHANNEL_NOTICE"] and self.registry["CHAT_MSG_CHANNEL_NOTICE"][self] then
				self:UnregisterEvent("CHAT_MSG_CHANNEL_NOTICE")
			end
			if self.registry["MEETINGSTONE_CHANGED"] and self.registry["MEETINGSTONE_CHANGED"][self] then
				self:UnregisterEvent("MEETINGSTONE_CHANGED")
			end
			if self.registry["MINIMAP_ZONE_CHANGED"] and self.registry["MINIMAP_ZONE_CHANGED"][self] then
				self:UnregisterEvent("MINIMAP_ZONE_CHANGED")
			end
			if self.registry["LANGUAGE_LIST_CHANGED"] and self.registry["LANGUAGE_LIST_CHANGED"][self] then
				self:UnregisterEvent("LANGUAGE_LIST_CHANGED")
			end
		end
		registeringFromAceEvent = true
		local f = function()
			self.playerLogin = true
			self:ScheduleEvent("AceEvent_FullyInitialized", func, 1)
		end
		self:RegisterEvent("MEETINGSTONE_CHANGED", f, true)
		self:RegisterEvent("CHAT_MSG_CHANNEL_NOTICE", function()
			self:ScheduleEvent("AceEvent_FullyInitialized", func, 0.05)
		end)
		self:RegisterEvent("LANGUAGE_LIST_CHANGED", function()
			if self.registry["MEETINGSTONE_CHANGED"] and self.registry["MEETINGSTONE_CHANGED"][self] then
				self:UnregisterEvent("MEETINGSTONE_CHANGED")
				self:RegisterEvent("MINIMAP_ZONE_CHANGED", f, true)
			end
		end)
		registeringFromAceEvent = nil
	end

	self.super.activate(self, oldLib, oldDeactivate)
	if oldLib then
		oldDeactivate(oldLib)
	end
end

AceLibrary:Register(AceEvent, MAJOR_VERSION, MINOR_VERSION, AceEvent.activate)
AceEvent = AceLibrary(MAJOR_VERSION)
```

## lib/AceHook/AceHook-2.1.lua
```lua
--[[
Name: AceHook-2.1
Revision: $Rev: 17638 $
Developed by: The Ace Development Team (http://www.wowace.com/index.php/The_Ace_Development_Team)
Inspired By: Ace 1.x by Turan (turan@gryphon.com)
Website: http://www.wowace.com/
Documentation: http://www.wowace.com/index.php/AceHook-2.1
SVN: http://svn.wowace.com/root/trunk/Ace2/AceHook-2.1
Description: Mixin to allow for safe hooking of functions, methods, and scripts.
Dependencies: AceLibrary, AceOO-2.0
]]

local MAJOR_VERSION = "AceHook-2.1"
local MINOR_VERSION = "$Revision: 17638 $"

-- This ensures the code is only executed if the libary doesn't already exist, or is a newer version
if not AceLibrary then error(MAJOR_VERSION .. " requires AceLibrary.") end
if not AceLibrary:IsNewVersion(MAJOR_VERSION, MINOR_VERSION) then return end

if loadstring("return function(...) return ... end") and AceLibrary:HasInstance(MAJOR_VERSION) then return end -- lua51 check
if not AceLibrary:HasInstance("AceOO-2.0") then error(MAJOR_VERSION .. " requires AceOO-2.0") end

--[[---------------------------------------------------------------------------------
  Create the library object
----------------------------------------------------------------------------------]]

local AceOO = AceLibrary:GetInstance("AceOO-2.0")
local AceHook = AceOO.Mixin {
								"Hook",
								"HookScript",
								"SecureHook",
								"Unhook",
								"UnhookAll",
								"HookReport",
								"IsHooked",
							}

--[[---------------------------------------------------------------------------------
  Library Definitions
----------------------------------------------------------------------------------]]

local protFuncs = {
	CameraOrSelectOrMoveStart = true, 	CameraOrSelectOrMoveStop = true,
	TurnOrActionStart = true,			TurnOrActionStop = true,
	PitchUpStart = true,				PitchUpStop = true,
	PitchDownStart = true,				PitchDownStop = true,
	MoveBackwardStart = true,			MoveBackwardStop = true,
	MoveForwardStart = true,			MoveForwardStop = true,
	Jump = true,						StrafeLeftStart = true,
	StrafeLeftStop = true,				StrafeRightStart = true,
	StrafeRightStop = true,				ToggleMouseMove = true,
	ToggleRun = true,					TurnLeftStart = true,
	TurnLeftStop = true,				TurnRightStart = true,
	TurnRightStop = true,
}

local function issecurevariable(x)
	if protFuncs[x] then
		return 1
	else
		return nil
	end
end

local _G = getfenv(0)

local function hooksecurefunc(arg1, arg2, arg3)
	if type(arg1) == "string" then
		arg1, arg2, arg3 = _G, arg1, arg2
	end
	local orig = arg1[arg2]
	arg1[arg2] = function(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
		local x1,x2,x3,x4,x5,x6,x7,x8,x9,x10,x11,x12,x13,x14,x15,x16,x17,x18,x19,x20 = orig(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
		
		arg3(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
		
		return x1,x2,x3,x4,x5,x6,x7,x8,x9,x10,x11,x12,x13,x14,x15,x16,x17,x18,x19,x20
	end
end

local protectedScripts = {
	OnClick = true,
}


local handlers, scripts, actives, registry

--[[---------------------------------------------------------------------------------
  Private definitions (Not exposed)
----------------------------------------------------------------------------------]]

local new, del
do
	local list = setmetatable({}, {__mode = "k"})
	function new()
		local t = next(list)
		if not t then
			return {}
		end
		list[t] = nil
		return t
	end
	
	function del(t)
		setmetatable(t, nil)
		for k in pairs(t) do
			t[k] = nil
		end
		list[t] = true
	end
end

local function createFunctionHook(self, func, handler, orig, secure)
	if not secure then
		if type(handler) == "string" then
			-- The handler is a method, need to self it
			local uid
			uid = function(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				if actives[uid] then
					return self[handler](self, a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				else
					return orig(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				end
			end
			return uid
		else
			-- The handler is a function, just call it
			local uid
			uid = function(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				if actives[uid] then
					return handler(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				else
					return orig(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				end
			end
			return uid
		end
	else
		-- secure hooks don't call the original method
		if type(handler) == "string" then
			-- The handler is a method, need to self it
			local uid
			uid = function(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				if actives[uid] then
					return self[handler](self, a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				end
			end
			return uid
		else
			-- The handler is a function, just call it
			local uid
			uid = function(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				if actives[uid] then
					return handler(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				end
			end
			return uid
		end
	end
end

local function createMethodHook(self, object, method, handler, orig, secure, script)
	if script then
		if type(handler) == "string" then
			local uid
			uid = function()
				if actives[uid] then
					return self[handler](self, object)
				else
					return orig()
				end
			end
			return uid
		else
			-- The handler is a function, just call it
			local uid
			uid = function()
				if actives[uid] then
					return handler(object)
				else
					return orig()
				end
			end
			return uid
		end
	elseif not secure then
		if type(handler) == "string" then
			local uid
			uid = function(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				if actives[uid] then
					return self[handler](self, a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				else
					return orig(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				end
			end
			return uid
		else
			-- The handler is a function, just call it
			local uid
			uid = function(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				if actives[uid] then
					return handler(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				else
					return orig(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				end
			end
			return uid
		end
	else
		-- secure hooks don't call the original method
		if type(handler) == "string" then
			local uid
			uid = function(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				if actives[uid] then
					return self[handler](self, a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				end
			end
			return uid
		else
			-- The handler is a function, just call it
			local uid
			uid = function(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				if actives[uid] then
					return handler(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12,a13,a14,a15,a16,a17,a18,a19,a20)
				end
			end
			return uid
		end
	end
end

local function hookFunction(self, func, handler, secure)
	local orig = _G[func]
	
	if not orig or type(orig) ~= "function" then
		AceHook:error("Attempt to hook a non-existant function %q", func)
	end
	
	if not handler then
		handler = func
	end

	if registry[self][func] then
		local uid = registry[self][func]
		
		if actives[uid] then
			-- We have an active hook from this source.  Don't multi-hook
			AceHook:error("%q already has an active hook from this source.", func)
		end
		
		if handlers[uid] == handler then
			-- The hook is inactive, so reactivate it
			actives[uid] = true
			return
		else
			AceHook:error("There is a stale hook for %q can't hook or reactivate.", func)
		end
	end
	
	if type(handler) == "string" then
		if type(self[handler]) ~= "function" then
			AceHook:error("Could not find the the handler %q when hooking function %q", handler, func)
		end
	elseif type(handler) ~= "function" then
		AceHook:error("Could not find the handler you supplied when hooking %q", func)
	end
	
	local uid = createFunctionHook(self, func, handler, orig, secure)
	registry[self][func] = uid
	actives[uid] = true
	handlers[uid] = handler
	
	if not secure then
		_G[func] = uid
		self.hooks[func] = orig
	else
		hooksecurefunc(func, uid)
	end
end

local function unhookFunction(self, func)
	if not registry[self][func] then
		AceHook:error("Tried to unhook %q which is not currently hooked.", func)
	end
	
	local uid = registry[self][func]
	
	if actives[uid] then
		-- See if we own the global function
		if self.hooks[func] and _G[func] == uid then
			_G[func] = self.hooks[func]
			self.hooks[func] = nil
			registry[self][func] = nil
			handlers[uid] = nil
			scripts[uid] = nil
			actives[uid] = nil
			-- Magically all-done
		else
			actives[uid] = nil
		end
	end
end

local function hookMethod(self, obj, method, handler, script, secure)
	if not handler then
		handler = method
	end
	
	if not obj or type(obj) ~= "table" then
		AceHook:error("The object you supplied could not be found, or isn't a table.")
	end
	
	local uid = registry[self][obj] and registry[self][obj][method]
	if uid then
		if actives[uid] then
			-- We have an active hook from this source.  Don't multi-hook
			AceHook:error("%q already has an active hook from this source.", method)
		end
		
		if handlers[uid] == handler then
			-- The hook is inactive, reactivate it.
			actives[uid] = true
			return
		else
			AceHook:error("There is a stale hook for %q can't hook or reactivate.", method)
		end
	end
	
	if type(handler) == "string" then
		if type(self[handler]) ~= "function" then
			AceHook:error("Could not find the handler %q you supplied when hooking method %q", handler, method)
		end
	elseif type(handler) ~= "function" then
		AceHook:error("Could not find the handler you supplied when hooking method %q", method)
	end
	
	local orig
	if script then
		if not obj.GetScript then
			AceHook:error("The object you supplied does not have a GetScript method.")
		end
		if not obj:HasScript(method) then
			AceHook:error("The object you supplied doesn't allow the %q method.", method)
		end
		
		orig = obj:GetScript(method)
		if type(orig) ~= "function" then
			-- Sometimes there is not a original function for a script.
			orig = function() end
		end
	else
		orig = obj[method]
	end
	if not orig then
		AceHook:error("Could not find the method or script %q you are trying to hook.", method)
	end
	
	if not registry[self][obj] then
		registry[self][obj] = new()
	end
	
	if not self.hooks[obj] then
		self.hooks[obj] = new()
	end
	
	local uid = createMethodHook(self, obj, method, handler, orig, secure, script)
	registry[self][obj][method] = uid
	actives[uid] = true
	handlers[uid] = handler
	scripts[uid] = script and true or nil
	
	if script then
		obj:SetScript(method, uid)
		self.hooks[obj][method] = orig
	elseif not secure then
		obj[method] = uid
		self.hooks[obj][method] = orig
	else
		hooksecurefunc(obj, method, uid)
	end
end

local function unhookMethod(self, obj, method)
	if not registry[self][obj] or not registry[self][obj][method] then
		AceHook:error("Attempt to unhook a method %q that is not currently hooked.", method)
		return
	end
	
	local uid = registry[self][obj][method]
	
	if actives[uid] then
		if scripts[uid] then -- If this is a script
			if obj:GetScript(method) == uid then
				-- We own the script.  Revert to normal.
				obj:SetScript(method, self.hooks[obj][method])
				self.hooks[obj][method] = nil
				registry[self][obj][method] = nil
				handlers[uid] = nil
				scripts[uid] = nil
				actives[uid] = nil
			else
				actives[uid] = nil
			end
		else
			if self.hooks[obj] and self.hooks[obj][method] and obj[method] == uid then
				-- We own the method.  Revert to normal.
				obj[method] = self.hooks[obj][method]
				self.hooks[obj][method] = nil
				registry[self][obj][method] = nil
				handlers[uid] = nil
				actives[uid] = nil
			else
				actives[uid] = nil
			end
		end
	end
	if self.hooks[obj] and not next(self.hooks[obj]) then
		self.hooks[obj] = del(self.hooks[obj])
	end
	if not next(registry[self][obj]) then
		registry[self][obj] = del(registry[self][obj])
	end
end

-- ("function" [, handler] [, hookSecure]) or (object, "method" [, handler] [, hookSecure])
function AceHook:Hook(object, method, handler, hookSecure)
	if type(object) == "string" then
		method, handler, hookSecure = object, method, handler
		if handler == true then
			handler, hookSecure = nil, true
		end
		if not hookSecure and issecurevariable(method) then
			AceHook:error("Attempt to hook secure function %q. Use `SecureHook' or add `true' to the argument list to override.", method)
		end
		AceHook:argCheck(handler, 3, "function", "string", "nil")
		AceHook:argCheck(hookSecure, 4, "boolean", "nil")
		hookFunction(self, method, handler, false)
	else
		if handler == true then
			handler, hookSecure = nil, true
		end
		if not hookSecure and issecurevariable(object, method) then
			AceHook:error("Attempt to hook secure method %q. Use `SecureHook' or add `true' to the argument list to override.", method)
		end
		AceHook:argCheck(object, 2, "table")
		AceHook:argCheck(method, 3, "string")
		AceHook:argCheck(handler, 4, "function", "string", "nil")
		AceHook:argCheck(hookSecure, 5, "boolean", "nil")
		hookMethod(self, object, method, handler, false, false)
	end
end

-- ("function", handler) or (object, "method", handler)
function AceHook:SecureHook(object, method, handler)
	if type(object) == "string" then
		method, handler = object, method
		AceHook:argCheck(handler, 3, "function", "string", "nil")
		hookFunction(self, method, handler, true)
	else
		AceHook:argCheck(object, 2, "table")
		AceHook:argCheck(method, 3, "string")
		AceHook:argCheck(handler, 4, "function", "string", "nil")
		hookMethod(self, object, method, handler, false, true)
	end
end

function AceHook:HookScript(frame, script, handler)
	AceHook:argCheck(frame, 2, "table")
	if not frame[0] or type(frame.IsFrameType) ~= "function" then
		AceHook:error("Bad argument #2 to `HookScript'. Expected frame.")
	end
	AceHook:argCheck(script, 3, "string")
	AceHook:argCheck(handler, 4, "function", "string", "nil")
	hookMethod(self, frame, script, handler, true, false)
end

-- ("function") or (object, "method")
function AceHook:IsHooked(obj, method)
	if type(obj) == "string" then
		if registry[self][obj] and actives[registry[self][obj]] then
			return true, handlers[registry[self][obj]]
		end
	else
		AceHook:argCheck(obj, 2, "string", "table")
		AceHook:argCheck(method, 3, "string")
		if registry[self][obj] and registry[self][obj][method] and actives[registry[self][obj][method]] then
			return true, handlers[registry[self][obj][method]]
		end
	end
	
	return false, nil
end

-- ("function") or (object, "method")
function AceHook:Unhook(obj, method)
	if type(obj) == "string" then
		unhookFunction(self, obj)
	else
		AceHook:argCheck(obj, 2, "string", "table")
		AceHook:argCheck(method, 3, "string")
		unhookMethod(self, obj, method)
	end
end

function AceHook:UnhookAll()
	for key, value in pairs(registry[self]) do
		if type(key) == "table" then
			for method in pairs(value) do
				self:Unhook(key, method)
			end
		else
			self:Unhook(key)
		end
	end
end

function AceHook:HookReport()
	DEFAULT_CHAT_FRAME:AddMessage("This is a list of all active hooks for this object:")
	if not next(registry[self]) then
		DEFAULT_CHAT_FRAME:AddMessage("No hooks")
	end
	
	for key, value in pairs(registry[self]) do
		if type(value) == "table" then
			for method, uid in pairs(value) do
				DEFAULT_CHAT_FRAME:AddMessage(string.format("object: %s method: %q |cff%s|r%s", tostring(key), method, actives[uid] and "00ff00Active" or "ffff00Inactive", not self.hooks[key][method] and " |cff7f7fff-Secure-|r" or ""))
			end
		else
			DEFAULT_CHAT_FRAME:AddMessage(string.format("function: %q |cff%s|r%s", tostring(key), actives[value] and "00ff00Active" or "ffff00Inactive", not self.hooks[key] and " |cff7f7fff-Secure-|r" or ""))
		end
	end
end

function AceHook:OnInstanceInit(object)
	if not object.hooks then
		object.hooks = new()
	end
	if not registry[object] then
		registry[object] = new()
	end
end

AceHook.OnManualEmbed = AceHook.OnInstanceInit

function AceHook:OnEmbedDisable(target)
	self.UnhookAll(target)
end

local function activate(self, oldLib, oldDeactivate)
	AceHook = self
	
	self.handlers = oldLib and oldLib.handlers or {}
	self.registry = oldLib and oldLib.registry or {}
	self.scripts = oldLib and oldLib.scripts or {}
	self.actives = oldLib and oldLib.actives or {}
	
	handlers = self.handlers
	registry = self.registry
	scripts = self.scripts
	actives = self.actives
	
	AceHook.super.activate(self, oldLib, oldDeactivate)
	
	if oldDeactivate then
		oldDeactivate(oldLib)
	end
end

AceLibrary:Register(AceHook, MAJOR_VERSION, MINOR_VERSION, activate)
```

## lib/AceLibrary/AceLibrary.lua
```lua
--[[
Name: AceLibrary
Revision: $Rev: 14130 $
Developed by: The Ace Development Team (http://www.wowace.com/index.php/The_Ace_Development_Team)
Inspired By: Iriel (iriel@vigilance-committee.org)
             Tekkub (tekkub@gmail.com)
             Revision: $Rev: 14130 $
Website: http://www.wowace.com/
Documentation: http://www.wowace.com/index.php/AceLibrary
SVN: http://svn.wowace.com/root/trunk/Ace2/AceLibrary
Description: Versioning library to handle other library instances, upgrading,
             and proper access.
             It also provides a base for libraries to work off of, providing
             proper error tools. It is handy because all the errors occur in the
             file that called it, not in the library file itself.
Dependencies: None
]]

local ACELIBRARY_MAJOR = "AceLibrary"
local ACELIBRARY_MINOR = "$Revision: 14130 $"

local table_setn
do
	local version = GetBuildInfo()
	if string.find(version, "^2%.") then
		-- 2.0.0
		table_setn = function() end
	else
		table_setn = table.setn
	end
end

local string_gfind = string.gmatch or string.gfind

local _G = getfenv(0)
local previous = _G[ACELIBRARY_MAJOR]
if previous and not previous:IsNewVersion(ACELIBRARY_MAJOR, ACELIBRARY_MINOR) then return end

-- @table AceLibrary
-- @brief System to handle all versioning of libraries.
local AceLibrary = {}
local AceLibrary_mt = {}
setmetatable(AceLibrary, AceLibrary_mt)

local tmp
local function error(self, message, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	if type(self) ~= "table" then
		_G.error(string.format("Bad argument #1 to `error' (table expected, got %s)", type(self)), 2)
	end
	if not tmp then
		tmp = {}
	else
		for k in pairs(tmp) do tmp[k] = nil end
		table_setn(tmp, 0)
	end

	table.insert(tmp, a1)
	table.insert(tmp, a2)
	table.insert(tmp, a3)
	table.insert(tmp, a4)
	table.insert(tmp, a5)
	table.insert(tmp, a6)
	table.insert(tmp, a7)
	table.insert(tmp, a8)
	table.insert(tmp, a9)
	table.insert(tmp, a10)
	table.insert(tmp, a11)
	table.insert(tmp, a12)
	table.insert(tmp, a13)
	table.insert(tmp, a14)
	table.insert(tmp, a15)
	table.insert(tmp, a16)
	table.insert(tmp, a17)
	table.insert(tmp, a18)
	table.insert(tmp, a19)
	table.insert(tmp, a20)

	local stack = debugstack()
	if not message then
		local _,_,second = string.find(stack, "\n(.-)\n")
		message = "error raised! " .. second
	else
		for i = 1,table.getn(tmp) do
			tmp[i] = tostring(tmp[i])
		end
		for i = 1,10 do
			table.insert(tmp, "nil")
		end
		message = string.format(message, unpack(tmp))
	end

	if getmetatable(self) and getmetatable(self).__tostring then
		message = string.format("%s: %s", tostring(self), message)
	elseif type(rawget(self, 'GetLibraryVersion')) == "function" and AceLibrary:HasInstance(self:GetLibraryVersion()) then
		message = string.format("%s: %s", self:GetLibraryVersion(), message)
	elseif type(rawget(self, 'class')) == "table" and type(rawget(self.class, 'GetLibraryVersion')) == "function" and AceLibrary:HasInstance(self.class:GetLibraryVersion()) then
		message = string.format("%s: %s", self.class:GetLibraryVersion(), message)
	end

	local first = string.gsub(stack, "\n.*", "")
	local file = string.gsub(first, ".*\\(.*).lua:%d+: .*", "%1")
	file = string.gsub(file, "([%(%)%.%*%+%-%[%]%?%^%$%%])", "%%%1")

	local i = 0
	for s in string_gfind(stack, "\n([^\n]*)") do
		i = i + 1
		if not string.find(s, file .. "%.lua:%d+:") then
			file = string.gsub(s, "^.*\\(.*).lua:%d+: .*", "%1")
			file = string.gsub(file, "([%(%)%.%*%+%-%[%]%?%^%$%%])", "%%%1")
			break
		end
	end
	local j = 0
	for s in string_gfind(stack, "\n([^\n]*)") do
		j = j + 1
		if j > i and not string.find(s, file .. "%.lua:%d+:") then
			_G.error(message, j + 1)
			return
		end
	end
	_G.error(message, 2)
	return
end

local function assert(self, condition, message, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	if not condition then
		if not message then
			local stack = debugstack()
			local _,_,second = string.find(stack, "\n(.-)\n")
			message = "assertion failed! " .. second
		end
		error(self, message, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
		return
	end
	return condition
end

local function argCheck(self, arg, num, kind, kind2, kind3, kind4, kind5)
	if type(num) ~= "number" then
		error(self, "Bad argument #3 to `argCheck' (number expected, got %s)", type(num))
	elseif type(kind) ~= "string" then
		error(self, "Bad argument #4 to `argCheck' (string expected, got %s)", type(kind))
	end
	local errored = false
	arg = type(arg)
	if arg ~= kind and arg ~= kind2 and arg ~= kind3 and arg ~= kind4 and arg ~= kind5 then
		local _,_,func = string.find(debugstack(), "`argCheck'.-([`<].-['>])")
		if not func then
			_,_,func = string.find(debugstack(), "([`<].-['>])")
		end
		if kind5 then
			error(self, "Bad argument #%s to %s (%s, %s, %s, %s, or %s expected, got %s)", tonumber(num) or 0/0, func, kind, kind2, kind3, kind4, kind5, arg)
		elseif kind4 then
			error(self, "Bad argument #%s to %s (%s, %s, %s, or %s expected, got %s)", tonumber(num) or 0/0, func, kind, kind2, kind3, kind4, arg)
		elseif kind3 then
			error(self, "Bad argument #%s to %s (%s, %s, or %s expected, got %s)", tonumber(num) or 0/0, func, kind, kind2, kind3, arg)
		elseif kind2 then
			error(self, "Bad argument #%s to %s (%s or %s expected, got %s)", tonumber(num) or 0/0, func, kind, kind2, arg)
		else
			error(self, "Bad argument #%s to %s (%s expected, got %s)", tonumber(num) or 0/0, func, kind, arg)
		end
	end
end

local function pcall(self, func, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20 = _G.pcall(func, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
	if not a1 then
		error(self, string.gsub(a2, ".-%.lua:%d-: ", ""))
	else
		return a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20
	end
end

local recurse = {}
local function addToPositions(t, major)
	if not AceLibrary.positions[t] or AceLibrary.positions[t] == major then
		rawset(t, recurse, true)
		AceLibrary.positions[t] = major
		for k,v in pairs(t) do
			if type(v) == "table" and not rawget(v, recurse) then
				addToPositions(v, major)
			end
			if type(k) == "table" and not rawget(k, recurse) then
				addToPositions(k, major)
			end
		end
		local mt = getmetatable(t)
		if mt and not rawget(mt, recurse) then
			addToPositions(mt, major)
		end
		rawset(t, recurse, nil)
	end
end

local function svnRevisionToNumber(text)
	if type(text) == "string" then
		if string.find(text, "^%$Revision: (%d+) %$$") then
			return tonumber((string.gsub(text, "^%$Revision: (%d+) %$$", "%1")))
		elseif string.find(text, "^%$Rev: (%d+) %$$") then
			return tonumber((string.gsub(text, "^%$Rev: (%d+) %$$", "%1")))
		elseif string.find(text, "^%$LastChangedRevision: (%d+) %$$") then
			return tonumber((string.gsub(text, "^%$LastChangedRevision: (%d+) %$$", "%1")))
		end
	elseif type(text) == "number" then
		return text
	end
	return nil
end

local crawlReplace
do
	local recurse = {}
	local function func(t, to, from)
		if recurse[t] then
			return
		end
		recurse[t] = true
		local mt = getmetatable(t)
		setmetatable(t, nil)
		rawset(t, to, rawget(t, from))
		rawset(t, from, nil)
		for k,v in pairs(t) do
			if v == from then
				t[k] = to
			elseif type(v) == "table" then
				if not recurse[v] then
					func(v, to, from)
				end
			end

			if type(k) == "table" then
				if not recurse[k] then
					func(k, to, from)
				end
			end
		end
		setmetatable(t, mt)
		if mt then
			if mt == from then
				setmetatable(t, to)
			elseif not recurse[mt] then
				func(mt, to, from)
			end
		end
	end
	function crawlReplace(t, to, from)
		func(t, to, from)
		for k in pairs(recurse) do
			recurse[k] = nil
		end
	end
end

-- @function destroyTable
-- @brief    remove all the contents of a table
-- @param t  table to destroy
local function destroyTable(t)
	setmetatable(t, nil)
	for k,v in pairs(t) do t[k] = nil end
	table_setn(t, 0)
end

local function isFrame(frame)
	return type(frame) == "table" and type(rawget(frame, 0)) == "userdata" and type(rawget(frame, 'IsFrameType')) == "function" and getmetatable(frame) and type(rawget(getmetatable(frame), '__index')) == "function"
end

local new, del
do
	local tables = setmetatable({}, {__mode = "k"})

	function new()
		local t = next(tables)
		if t then
			tables[t] = nil
			return t
		else
			return {}
		end
	end

	function del(t, depth)
		if depth and depth > 0 then
			for k,v in pairs(t) do
				if type(v) == "table" and not isFrame(v) then
					del(v, depth - 1)
				end
			end
		end
		destroyTable(t)
		tables[t] = true
	end
end

-- @function   copyTable
-- @brief      Create a shallow copy of a table and return it.
-- @param from The table to copy from
-- @return     A shallow copy of the table
local function copyTable(from)
	local to = new()
	for k,v in pairs(from) do to[k] = v end
	table_setn(to, table.getn(from))
	setmetatable(to, getmetatable(from))
	return to
end

-- @function         deepTransfer
-- @brief            Fully transfer all data, keeping proper previous table
--                   backreferences stable.
-- @param to         The table with which data is to be injected into
-- @param from       The table whose data will be injected into the first
-- @param saveFields If available, a shallow copy of the basic data is saved
--                   in here.
-- @param list       The account of table references
-- @param list2      The current status on which tables have been traversed.
local deepTransfer
do
	-- @function   examine
	-- @brief      Take account of all the table references to be shared
	--             between the to and from tables.
	-- @param to   The table with which data is to be injected into
	-- @param from The table whose data will be injected into the first
	-- @param list An account of the table references
	local function examine(to, from, list, major)
		list[from] = to
		for k,v in pairs(from) do
			if rawget(to, k) and type(from[k]) == "table" and type(to[k]) == "table" and not list[from[k]] then
				if from[k] == to[k] then
					list[from[k]] = to[k]
				elseif AceLibrary.positions[from[v]] ~= major and AceLibrary.positions[from[v]] then
					list[from[k]] = from[k]
				elseif not list[from[k]] then
					examine(to[k], from[k], list, major)
				end
			end
		end
		return list
	end

	function deepTransfer(to, from, saveFields, major, list, list2)
		setmetatable(to, nil)
		local createdList
		if not list then
			createdList = true
			list = new()
			list2 = new()
			examine(to, from, list, major)
		end
		list2[to] = to
		for k,v in pairs(to) do
			if type(rawget(from, k)) ~= "table" or type(v) ~= "table" or isFrame(v) then
				if saveFields then
					saveFields[k] = v
				end
				to[k] = nil
			elseif v ~= _G then
				if saveFields then
					saveFields[k] = copyTable(v)
				end
			end
		end
		for k in pairs(from) do
			if rawget(to, k) and to[k] ~= from[k] and AceLibrary.positions[to[k]] == major and from[k] ~= _G then
				if not list2[to[k]] then
					deepTransfer(to[k], from[k], nil, major, list, list2)
				end
				to[k] = list[to[k]] or list2[to[k]]
			else
				rawset(to, k, from[k])
			end
		end
		table_setn(to, table.getn(from))
		setmetatable(to, getmetatable(from))
		local mt = getmetatable(to)
		if mt then
			if list[mt] then
				setmetatable(to, list[mt])
			elseif mt.__index and list[mt.__index] then
				mt.__index = list[mt.__index]
			end
		end
		destroyTable(from)
		if createdList then
			del(list)
			del(list2)
		end
	end
end

-- @method      TryToLoadStandalone
-- @brief       Attempt to find and load a standalone version of the requested library
-- @param major A string representing the major version
-- @return      If library is found, return values from the call to LoadAddOn are returned
--              If the library has been requested previously, nil is returned.
local function TryToLoadStandalone(major)
	if not AceLibrary.scannedlibs then AceLibrary.scannedlibs = {} end
	if AceLibrary.scannedlibs[major] then return end

	AceLibrary.scannedlibs[major] = true

	local name, _, _, enabled, loadable = GetAddOnInfo(major)
	if loadable then
		return LoadAddOn(name)
	end

	for i=1,GetNumAddOns() do
		if GetAddOnMetadata(i, "X-AceLibrary-"..major) then
			local name, _, _, enabled, loadable = GetAddOnInfo(i)
			if loadable then
				return LoadAddOn(name)
			end
		end
	end
end

-- @method      IsNewVersion
-- @brief       Obtain whether the supplied version would be an upgrade to the
--              current version. This allows for bypass code in library
--              declaration.
-- @param major A string representing the major version
-- @param minor An integer or an svn revision string representing the minor version
-- @return      whether the supplied version would be newer than what is
--              currently available.
function AceLibrary:IsNewVersion(major, minor)
	argCheck(self, major, 2, "string")
	TryToLoadStandalone(major)

	if type(minor) == "string" then
		local m = svnRevisionToNumber(minor)
		if m then
			minor = m
		else
			_G.error(string.format("Bad argument #3 to  `IsNewVersion'. Must be a number or SVN revision string. %q is not appropriate", minor), 2)
		end
	end
	argCheck(self, minor, 3, "number")
	local data = self.libs[major]
	if not data then
		return true
	end
	return data.minor < minor
end

-- @method      HasInstance
-- @brief       Returns whether an instance exists. This allows for optional support of a library.
-- @param major A string representing the major version.
-- @param minor (optional) An integer or an svn revision string representing the minor version.
-- @return      Whether an instance exists.
function AceLibrary:HasInstance(major, minor)
	argCheck(self, major, 2, "string")
	TryToLoadStandalone(major)

	if minor then
		if type(minor) == "string" then
			local m = svnRevisionToNumber(minor)
			if m then
				minor = m
			else
				_G.error(string.format("Bad argument #3 to  `HasInstance'. Must be a number or SVN revision string. %q is not appropriate", minor), 2)
			end
		end
		argCheck(self, minor, 3, "number")
		if not self.libs[major] then
			return
		end
		return self.libs[major].minor == minor
	end
	return self.libs[major] and true
end

-- @method      GetInstance
-- @brief       Returns the library with the given major/minor version.
-- @param major A string representing the major version.
-- @param minor (optional) An integer or an svn revision string representing the minor version.
-- @return      The library with the given major/minor version.
function AceLibrary:GetInstance(major, minor)
	argCheck(self, major, 2, "string")
	TryToLoadStandalone(major)

	local data = self.libs[major]
	if not data then
		_G.error(string.format("Cannot find a library instance of %s.", major), 2)
		return
	end
	if minor then
		if type(minor) == "string" then
			local m = svnRevisionToNumber(minor)
			if m then
				minor = m
			else
				_G.error(string.format("Bad argument #3 to  `GetInstance'. Must be a number or SVN revision string. %q is not appropriate", minor), 2)
			end
		end
		argCheck(self, minor, 2, "number")
		if data.minor ~= minor then
			_G.error(string.format("Cannot find a library instance of %s, minor version %d.", major, minor), 2)
			return
		end
	end
	return data.instance
end

-- Syntax sugar.  AceLibrary("FooBar-1.0")
AceLibrary_mt.__call = AceLibrary.GetInstance

local donothing

local AceEvent

-- @method               Register
-- @brief                Registers a new version of a given library.
-- @param newInstance    the library to register
-- @param major          the major version of the library
-- @param minor          the minor version of the library
-- @param activateFunc   (optional) A function to be called when the library is
--                       fully activated. Takes the arguments
--                       (newInstance [, oldInstance, oldDeactivateFunc]). If
--                       oldInstance is given, you should probably call
--                       oldDeactivateFunc(oldInstance).
-- @param deactivateFunc (optional) A function to be called by a newer library's
--                       activateFunc.
-- @param externalFunc   (optional) A function to be called whenever a new
--                       library is registered.
function AceLibrary:Register(newInstance, major, minor, activateFunc, deactivateFunc, externalFunc)
	argCheck(self, newInstance, 2, "table")
	argCheck(self, major, 3, "string")
	if type(minor) == "string" then
		local m = svnRevisionToNumber(minor)
		if m then
			minor = m
		else
			_G.error(string.format("Bad argument #4 to  `Register'. Must be a number or SVN revision string. %q is not appropriate", minor), 2)
		end
	end
	argCheck(self, minor, 4, "number")
	if math.floor(minor) ~= minor or minor < 0 then
		error(self, "Bad argument #4 to `Register' (integer >= 0 expected, got %s)", minor)
	end
	argCheck(self, activateFunc, 5, "function", "nil")
	argCheck(self, deactivateFunc, 6, "function", "nil")
	argCheck(self, externalFunc, 7, "function", "nil")
	if not deactivateFunc then
		if not donothing then
			donothing = function() end
		end
		deactivateFunc = donothing
	end
	local data = self.libs[major]
	if not data then
		-- This is new
		local instance = copyTable(newInstance)
		crawlReplace(instance, instance, newInstance)
		destroyTable(newInstance)
		if AceLibrary == newInstance then
			self = instance
			AceLibrary = instance
		end
		self.libs[major] = {
			instance = instance,
			minor = minor,
			deactivateFunc = deactivateFunc,
			externalFunc = externalFunc,
		}
		rawset(instance, 'GetLibraryVersion', function(self)
			return major, minor
		end)
		if not rawget(instance, 'error') then
			rawset(instance, 'error', error)
		end
		if not rawget(instance, 'assert') then
			rawset(instance, 'assert', assert)
		end
		if not rawget(instance, 'argCheck') then
			rawset(instance, 'argCheck', argCheck)
		end
		if not rawget(instance, 'pcall') then
			rawset(instance, 'pcall', pcall)
		end
		addToPositions(instance, major)
		if activateFunc then
			activateFunc(instance, nil, nil) -- no old version, so explicit nil
		end

		if externalFunc then
			for k,data in pairs(self.libs) do
				if k ~= major then
					externalFunc(instance, k, data.instance)
				end
			end
		end

		for k,data in pairs(self.libs) do
			if k ~= major and data.externalFunc then
				data.externalFunc(data.instance, major, instance)
			end
		end
		if major == "AceEvent-2.0" then
			AceEvent = instance
		end
		if AceEvent then
			AceEvent.TriggerEvent(self, "AceLibrary_Register", major, instance)
		end

		return instance
	end
	local instance = data.instance
	if minor <= data.minor then
		-- This one is already obsolete, raise an error.
		_G.error(string.format("Obsolete library registered. %s is already registered at version %d. You are trying to register version %d. Hint: if not AceLibrary:IsNewVersion(%q, %d) then return end", major, data.minor, minor, major, minor), 2)
		return
	end
	-- This is an update
	local oldInstance = new()

	addToPositions(newInstance, major)
	local isAceLibrary = (AceLibrary == newInstance)
	local old_error, old_assert, old_argCheck, old_pcall
	if isAceLibrary then
		self = instance
		AceLibrary = instance

		old_error = instance.error
		old_assert = instance.assert
		old_argCheck = instance.argCheck
		old_pcall = instance.pcall

		self.error = error
		self.assert = assert
		self.argCheck = argCheck
		self.pcall = pcall
	end
	deepTransfer(instance, newInstance, oldInstance, major)
	crawlReplace(instance, instance, newInstance)
	local oldDeactivateFunc = data.deactivateFunc
	data.minor = minor
	data.deactivateFunc = deactivateFunc
	data.externalFunc = externalFunc
	rawset(instance, 'GetLibraryVersion', function(self)
		return major, minor
	end)
	if not rawget(instance, 'error') then
		rawset(instance, 'error', error)
	end
	if not rawget(instance, 'assert') then
		rawset(instance, 'assert', assert)
	end
	if not rawget(instance, 'argCheck') then
		rawset(instance, 'argCheck', argCheck)
	end
	if not rawget(instance, 'pcall') then
		rawset(instance, 'pcall', pcall)
	end
	if isAceLibrary then
		for _,v in pairs(self.libs) do
			local i = type(v) == "table" and v.instance
			if type(i) == "table" then
				if not rawget(i, 'error') or i.error == old_error then
					rawset(i, 'error', error)
				end
				if not rawget(i, 'assert') or i.assert == old_assert then
					rawset(i, 'assert', assert)
				end
				if not rawget(i, 'argCheck') or i.argCheck == old_argCheck then
					rawset(i, 'argCheck', argCheck)
				end
				if not rawget(i, 'pcall') or i.pcall == old_pcall then
					rawset(i, 'pcall', pcall)
				end
			end
		end
	end
	if activateFunc then
		activateFunc(instance, oldInstance, oldDeactivateFunc)
	else
		oldDeactivateFunc(oldInstance)
	end
	del(oldInstance)

	if externalFunc then
		for k,data in pairs(self.libs) do
			if k ~= major then
				externalFunc(instance, k, data.instance)
			end
		end
	end

	return instance
end

local iter
function AceLibrary:IterateLibraries()
	if not iter then
		local function iter(t, k)
			k = next(t, k)
			if not k then
				return nil
			else
				return k, t[k].instance
			end
		end
	end
	return iter, self.libs, nil
end

-- @function            Activate
-- @brief               The activateFunc for AceLibrary itself. Called when
--                      AceLibrary properly registers.
-- @param self          Reference to AceLibrary
-- @param oldLib        (optional) Reference to an old version of AceLibrary
-- @param oldDeactivate (optional) Function to deactivate the old lib
local function activate(self, oldLib, oldDeactivate)
	if not self.libs then
		if oldLib then
			self.libs = oldLib.libs
			self.scannedlibs = oldLib.scannedlibs
		end
		if not self.libs then
			self.libs = {}
		end
		if not self.scannedlibs then
			self.scannedlibs = {}
		end
	end
	if not self.positions then
		if oldLib then
			self.positions = oldLib.positions
		end
		if not self.positions then
			self.positions = setmetatable({}, { __mode = "k" })
		end
	end

	-- Expose the library in the global environment
	_G[ACELIBRARY_MAJOR] = self

	if oldDeactivate then
		oldDeactivate(oldLib)
	end
end

if not previous then
	previous = AceLibrary
end
if not previous.libs then
	previous.libs = {}
end
AceLibrary.libs = previous.libs
if not previous.positions then
	previous.positions = setmetatable({}, { __mode = "k" })
end
AceLibrary.positions = previous.positions
AceLibrary:Register(AceLibrary, ACELIBRARY_MAJOR, ACELIBRARY_MINOR, activate)
```

## lib/AceLocale/AceLocale-2.2.lua
```lua
--[[
Name: AceLocale-2.2
Revision: $Rev: 15291 $
Developed by: The Ace Development Team (http://www.wowace.com/index.php/The_Ace_Development_Team)
Inspired By: Ace 1.x by Turan (turan@gryphon.com)
Website: http://www.wowace.com/
Documentation: http://www.wowace.com/index.php/AceLocale-2.2
SVN: http://svn.wowace.com/root/trunk/Ace2/AceLocale-2.2
Description: Localization library for addons to use to handle proper
             localization and internationalization.
Dependencies: AceLibrary
]]

local MAJOR_VERSION = "AceLocale-2.2"
local MINOR_VERSION = "$Revision: 15291 $"

if not AceLibrary then error(MAJOR_VERSION .. " requires AceLibrary.") end
if not AceLibrary:IsNewVersion(MAJOR_VERSION, MINOR_VERSION) then return end

local AceLocale = {}

local DEFAULT_LOCALE = "enUS"
local _G = getfenv(0)

local BASE_TRANSLATIONS, DEBUGGING, TRANSLATIONS, BASE_LOCALE, TRANSLATION_TABLES, REVERSE_TRANSLATIONS, STRICTNESS, DYNAMIC_LOCALES, CURRENT_LOCALE, NAME

local rawget = rawget
local rawset = rawset
local type = type

local newRegistries = {}
local scheduleClear

local lastSelf
local __index = function(self, key)
	lastSelf = self
	local value = (rawget(self, TRANSLATIONS) or AceLocale.prototype)[key]
	rawset(self, key, value)
	return value
end

local __newindex = function(self, k, v)
	if type(v) ~= "function" and type(k) ~= "table" then
		AceLocale.error(self, "Cannot change the values of an AceLocale instance.")
	end
	rawset(self, k, v)
end

local __tostring = function(self)
	if type(rawget(self, 'GetLibraryVersion')) == "function" then
		return self:GetLibraryVersion()
	else
		return "AceLocale(" .. self[NAME] .. ")"
	end
end

local function clearCache(self)
	if not rawget(self, BASE_TRANSLATIONS) then
		return
	end
	
	local cache = self[BASE_TRANSLATIONS]
	rawset(self, REVERSE_TRANSLATIONS, nil)
	
	for k in pairs(self) do
		if rawget(cache, k) ~= nil then
			self[k] = nil
		end
	end
	rawset(self, 'tmp', true)
	self.tmp = nil
end

local function refixInstance(instance)
	if getmetatable(instance) then
		setmetatable(instance, nil)
	end
	local translations = instance[TRANSLATIONS]
	if translations then
		if getmetatable(translations) then
			setmetatable(translations, nil)
		end
		local baseTranslations = instance[BASE_TRANSLATIONS]
		if getmetatable(baseTranslations) then
			setmetatable(baseTranslations, nil)
		end
		if translations == baseTranslations or instance[STRICTNESS] then
			setmetatable(instance, {
				__index = __index,
				__newindex = __newindex,
				__tostring = __tostring
			})
			
			setmetatable(translations, {
				__index = AceLocale.prototype
			})
		else
			setmetatable(instance, {
				__index = __index,
				__newindex = __newindex,
				__tostring = __tostring
			})
			
			setmetatable(translations, {
				__index = baseTranslations,
			})
			
			setmetatable(baseTranslations, {
				__index = AceLocale.prototype,
			})
		end
	else
		setmetatable(instance, {
			__index = __index,
			__newindex = __newindex,
			__tostring = __tostring,
		})
	end
	clearCache(instance)
	newRegistries[instance] = true
	scheduleClear()
	return instance
end

function AceLocale:new(name)
	self:argCheck(name, 2, "string")
	
	if self.registry[name] and type(rawget(self.registry[name], 'GetLibraryVersion')) ~= "function" then
		return self.registry[name]
	end
	
	AceLocale.registry[name] = refixInstance({
		[STRICTNESS] = false,
		[NAME] = name,
	})
	newRegistries[AceLocale.registry[name]] = true
	return AceLocale.registry[name]
end

AceLocale.prototype = { class = AceLocale }

function AceLocale.prototype:EnableDebugging()
	if rawget(self, BASE_TRANSLATIONS) then
		AceLocale.error(self, "Cannot enable debugging after a translation has been registered.")
	end
	rawset(self, DEBUGGING, true)
end

function AceLocale.prototype:EnableDynamicLocales(override)
	AceLocale.argCheck(self, override, 2, "boolean", "nil")
	if not override and rawget(self, BASE_TRANSLATIONS) then
		AceLocale.error(self, "Cannot enable dynamic locales after a translation has been registered.")
	end
	if not rawget(self, DYNAMIC_LOCALES) then
		rawset(self, DYNAMIC_LOCALES, true)
		if rawget(self, BASE_LOCALE) then
			if not rawget(self, TRANSLATION_TABLES) then
				rawset(self, TRANSLATION_TABLES, {})
			end
			self[TRANSLATION_TABLES][self[BASE_LOCALE]] = self[BASE_TRANSLATIONS]
			self[TRANSLATION_TABLES][self[CURRENT_LOCALE]] = self[TRANSLATIONS]
		end
	end
end

function AceLocale.prototype:RegisterTranslations(locale, func)
	AceLocale.argCheck(self, locale, 2, "string")
	AceLocale.argCheck(self, func, 3, "function")
	
	if locale == rawget(self, BASE_LOCALE) then
		AceLocale.error(self, "Cannot provide the same locale more than once. %q provided twice.", locale)
	end
	
	if rawget(self, BASE_TRANSLATIONS) and GetLocale() ~= locale then
		if rawget(self, DEBUGGING) or rawget(self, DYNAMIC_LOCALES) then
			if not rawget(self, TRANSLATION_TABLES) then
				rawset(self, TRANSLATION_TABLES, {})
			end
			if self[TRANSLATION_TABLES][locale] then
				AceLocale.error(self, "Cannot provide the same locale more than once. %q provided twice.", locale)
			end
			local t = func()
			func = nil
			if type(t) ~= "table" then
				AceLocale.error(self, "Bad argument #3 to `RegisterTranslations'. function did not return a table. (expected table, got %s)", type(t))
			end
			self[TRANSLATION_TABLES][locale] = t
			t = nil
		end
		func = nil
		return
	end
	local t = func()
	func = nil
	if type(t) ~= "table" then
		AceLocale.error(self, "Bad argument #3 to `RegisterTranslations'. function did not return a table. (expected table, got %s)", type(t))
	end
	
	rawset(self, TRANSLATIONS, t)
	if not rawget(self, BASE_TRANSLATIONS) then
		rawset(self, BASE_TRANSLATIONS, t)
		rawset(self, BASE_LOCALE, locale)
		for key,value in pairs(t) do
			if value == true then
				t[key] = key
			end
		end
	else
		for key, value in pairs(self[TRANSLATIONS]) do
			if not rawget(self[BASE_TRANSLATIONS], key) then
				AceLocale.error(self, "Improper translation exists. %q is likely misspelled for locale %s.", key, locale)
			end
			if value == true then
				AceLocale.error(self, "Can only accept true as a value on the base locale. %q is the base locale, %q is not.", rawget(self, BASE_LOCALE), locale)
			end
		end
	end
	rawset(self, CURRENT_LOCALE, locale)
	refixInstance(self)
	if rawget(self, DEBUGGING) or rawget(self, DYNAMIC_LOCALES) then
		if not rawget(self, TRANSLATION_TABLES) then
			rawset(self, TRANSLATION_TABLES, {})
		end
		self[TRANSLATION_TABLES][locale] = t
	end
	t = nil
end

function AceLocale.prototype:SetLocale(locale)
	AceLocale.argCheck(self, locale, 2, "string", "boolean")
	if not rawget(self, DYNAMIC_LOCALES) then
		AceLocale.error(self, "Cannot call `SetLocale' without first calling `EnableDynamicLocales'.")
	end
	if not rawget(self, TRANSLATION_TABLES) then
		AceLocale.error(self, "Cannot call `SetLocale' without first calling `RegisterTranslations'.")
	end
	if locale == true then
		locale = GetLocale()
		if not self[TRANSLATION_TABLES][locale] then
			locale = self[BASE_LOCALE]
		end
	end
	
	if self[CURRENT_LOCALE] == locale then
		return
	end
	
	if not self[TRANSLATION_TABLES][locale] then
		AceLocale.error(self, "Locale %q not registered.", locale)
	end
	
	self[TRANSLATIONS] = self[TRANSLATION_TABLES][locale]
	self[CURRENT_LOCALE] = locale
	refixInstance(self)
end

function AceLocale.prototype:GetLocale()
	if not rawget(self, TRANSLATION_TABLES) then
		AceLocale.error(self, "Cannot call `GetLocale' without first calling `RegisterTranslations'.")
	end
	return self[CURRENT_LOCALE]
end

local function iter(t, position)
	return (next(t, position))
end

function AceLocale.prototype:IterateAvailableLocales()
	if not rawget(self, DYNAMIC_LOCALES) then
		AceLocale.error(self, "Cannot call `IterateAvailableLocales' without first calling `EnableDynamicLocales'.")
	end
	if not rawget(self, TRANSLATION_TABLES) then
		AceLocale.error(self, "Cannot call `IterateAvailableLocales' without first calling `RegisterTranslations'.")
	end
	return iter, self[TRANSLATION_TABLES], nil
end

function AceLocale.prototype:HasLocale(locale)
	if not rawget(self, DYNAMIC_LOCALES) then
		AceLocale.error(self, "Cannot call `HasLocale' without first calling `EnableDynamicLocales'.")
	end
	AceLocale.argCheck(self, locale, 2, "string")
	return rawget(self, TRANSLATION_TABLES) and self[TRANSLATION_TABLES][locale] ~= nil
end

function AceLocale.prototype:SetStrictness(strict)
	AceLocale.argCheck(self, strict, 2, "boolean")
	local mt = getmetatable(self)
	if not mt then
		AceLocale.error(self, "Cannot call `SetStrictness' without a metatable.")
	end
	if not rawget(self, TRANSLATIONS) then
		AceLocale.error(self, "No translations registered.")
	end
	rawset(self, STRICTNESS, strict)
	refixInstance(self)
end

local function initReverse(self)
	rawset(self, REVERSE_TRANSLATIONS, {})
	local alpha = self[TRANSLATIONS]
	local bravo = self[REVERSE_TRANSLATIONS]
	for base, localized in pairs(alpha) do
		bravo[localized] = base
	end
end

function AceLocale.prototype:GetTranslation(text)
	AceLocale.argCheck(self, text, 1, "string", "number")
	if not rawget(self, TRANSLATIONS) then
		AceLocale.error(self, "No translations registered")
	end
	return self[text]
end

function AceLocale.prototype:GetStrictTranslation(text)
	AceLocale.argCheck(self, text, 1, "string", "number")
	local x = rawget(self, TRANSLATIONS)
	if not x then
		AceLocale.error(self, "No translations registered")
	end
	local value = rawget(x, text)
	if value == nil then
		AceLocale.error(self, "Translation %q does not exist for locale %s", text, self[CURRENT_LOCALE])
	end
	return value
end

function AceLocale.prototype:GetReverseTranslation(text)
	local x = rawget(self, REVERSE_TRANSLATIONS)
	if not x then
		if not rawget(self, TRANSLATIONS) then
			AceLocale.error(self, "No translations registered")
		end
		initReverse(self)
		x = self[REVERSE_TRANSLATIONS]
	end
	local translation = x[text]
	if not translation then
		AceLocale.error(self, "Reverse translation for %q does not exist", text)
	end
	return translation
end

function AceLocale.prototype:GetIterator()
	local x = rawget(self, TRANSLATIONS)
	if not x then
		AceLocale.error(self, "No translations registered")
	end
	return next, x, nil
end

function AceLocale.prototype:GetReverseIterator()
	local x = rawget(self, REVERSE_TRANSLATIONS)
	if not x then
		if not rawget(self, TRANSLATIONS) then
			AceLocale.error(self, "No translations registered")
		end
		initReverse(self)
		x = self[REVERSE_TRANSLATIONS]
	end
	return next, x, nil
end

function AceLocale.prototype:HasTranslation(text)
	AceLocale.argCheck(self, text, 1, "string", "number")
	local x = rawget(self, TRANSLATIONS)
	if not x then
		AceLocale.error(self, "No translations registered")
	end
	return rawget(x, text) and true
end

function AceLocale.prototype:HasReverseTranslation(text)
	local x = rawget(self, REVERSE_TRANSLATIONS)
	if not x then
		if not rawget(self, TRANSLATIONS) then
			AceLocale.error(self, "No translations registered")
		end
		initReverse(self)
		x = self[REVERSE_TRANSLATIONS]
	end
	return x[text] and true
end

function AceLocale.prototype:Debug()
	if not rawget(self, DEBUGGING) then
		return
	end
	local words = {}
	local locales = {"enUS", "deDE", "frFR", "koKR", "zhCN", "zhTW", "esES"}
	local localizations = {}
	DEFAULT_CHAT_FRAME:AddMessage("--- AceLocale Debug ---")
	for _,locale in ipairs(locales) do
		if not self[TRANSLATION_TABLES][locale] then
			DEFAULT_CHAT_FRAME:AddMessage(string.format("Locale %q not found", locale))
		else
			localizations[locale] = self[TRANSLATION_TABLES][locale]
		end
	end
	local localeDebug = {}
	for locale, localization in pairs(localizations) do
		localeDebug[locale] = {}
		for word in pairs(localization) do
			if type(localization[word]) == "table" then
				if type(words[word]) ~= "table" then
					words[word] = {}
				end
				for bit in pairs(localization[word]) do
					if type(localization[word][bit]) == "string" then
						words[word][bit] = true
					end
				end
			elseif type(localization[word]) == "string" then
				words[word] = true
			end
		end
	end
	for word in pairs(words) do
		if type(words[word]) == "table" then
			for bit in pairs(words[word]) do
				for locale, localization in pairs(localizations) do
					if not rawget(localization, word) or not localization[word][bit] then
						localeDebug[locale][word .. "::" .. bit] = true
					end
				end
			end
		else
			for locale, localization in pairs(localizations) do
				if not rawget(localization, word) then
					localeDebug[locale][word] = true
				end
			end
		end
	end
	for locale, t in pairs(localeDebug) do
		if not next(t) then
			DEFAULT_CHAT_FRAME:AddMessage(string.format("Locale %q complete", locale))
		else
			DEFAULT_CHAT_FRAME:AddMessage(string.format("Locale %q missing:", locale))
			for word in pairs(t) do
				DEFAULT_CHAT_FRAME:AddMessage(string.format("    %q", word))
			end
		end
	end
	DEFAULT_CHAT_FRAME:AddMessage("--- End AceLocale Debug ---")
end

setmetatable(AceLocale.prototype, {
	__index = function(self, k)
		if type(k) ~= "table" and k ~= 0 and k ~= "GetLibraryVersion"  and k ~= "error" and k ~= "assert" and k ~= "argCheck" and k ~= "pcall" then -- HACK: remove "GetLibraryVersion" and such later.
			AceLocale.error(lastSelf or self, "Translation %q does not exist.", k)
		end
		return nil
	end
})

local function activate(self, oldLib, oldDeactivate)
	AceLocale = self
	
	self.frame = oldLib and oldLib.frame or CreateFrame("Frame")
	self.registry = oldLib and oldLib.registry or {}
	self.BASE_TRANSLATIONS = oldLib and oldLib.BASE_TRANSLATIONS or {}
	self.DEBUGGING = oldLib and oldLib.DEBUGGING or {}
	self.TRANSLATIONS = oldLib and oldLib.TRANSLATIONS or {}
	self.BASE_LOCALE = oldLib and oldLib.BASE_LOCALE or {}
	self.TRANSLATION_TABLES = oldLib and oldLib.TRANSLATION_TABLES or {}
	self.REVERSE_TRANSLATIONS = oldLib and oldLib.REVERSE_TRANSLATIONS or {}
	self.STRICTNESS = oldLib and oldLib.STRICTNESS or {}
	self.NAME = oldLib and oldLib.NAME or {}
	self.DYNAMIC_LOCALES = oldLib and oldLib.DYNAMIC_LOCALES or {}
	self.CURRENT_LOCALE = oldLib and oldLib.CURRENT_LOCALE or {}
	
	BASE_TRANSLATIONS = self.BASE_TRANSLATIONS
	DEBUGGING = self.DEBUGGING
	TRANSLATIONS = self.TRANSLATIONS
	BASE_LOCALE = self.BASE_LOCALE
	TRANSLATION_TABLES = self.TRANSLATION_TABLES
	REVERSE_TRANSLATIONS = self.REVERSE_TRANSLATIONS
	STRICTNESS = self.STRICTNESS
	NAME = self.NAME
	DYNAMIC_LOCALES = self.DYNAMIC_LOCALES
	CURRENT_LOCALE = self.CURRENT_LOCALE
	
	
	local GetTime = GetTime
	local timeUntilClear = GetTime() + 5
	scheduleClear = function()
		if next(newRegistries) then
			self.frame:Show()
			timeUntilClear = GetTime() + 5
		end
	end
	
	if not self.registry then
		self.registry = {}
	else
		for name, instance in pairs(self.registry) do
			local name = name
			local mt = getmetatable(instance)
			setmetatable(instance, nil)
			instance[NAME] = name
			local strict
			if instance[STRICTNESS] ~= nil then
				strict = instance[STRICTNESS]
			elseif instance[TRANSLATIONS] ~= instance[BASE_TRANSLATIONS] then
				if getmetatable(instance[TRANSLATIONS]).__index == oldLib.prototype then
					strict = true
				end
			end
			instance[STRICTNESS] = strict and true or false
			refixInstance(instance)
		end
	end
	
	self.frame:SetScript("OnEvent", scheduleClear)
	self.frame:SetScript("OnUpdate", function() -- (this, elapsed)
		if timeUntilClear - GetTime() <= 0 then
			self.frame:Hide()
			for k in pairs(newRegistries) do
				clearCache(k)
				newRegistries[k] = nil
				k = nil
			end
		end
	end)
	self.frame:UnregisterAllEvents()
	self.frame:RegisterEvent("ADDON_LOADED")
	self.frame:RegisterEvent("PLAYER_ENTERING_WORLD")
	self.frame:Show()
	
	if oldDeactivate then
		oldDeactivate(oldLib)
	end
end

AceLibrary:Register(AceLocale, MAJOR_VERSION, MINOR_VERSION, activate)
```

## lib/AceOO/AceOO-2.0.lua
```lua
--[[
Name: AceOO-2.0
Revision: $Rev: 11577 $
Developed by: The Ace Development Team (http://www.wowace.com/index.php/The_Ace_Development_Team)
Inspired By: Ace 1.x by Turan (turan@gryphon.com)
Website: http://www.wowace.com/
Documentation: http://www.wowace.com/index.php/AceOO-2.0
SVN: http://svn.wowace.com/root/trunk/Ace2/AceOO-2.0
Description: Library to provide an object-orientation framework.
Dependencies: AceLibrary
]]

local MAJOR_VERSION = "AceOO-2.0"
local MINOR_VERSION = "$Revision: 11577 $"

-- This ensures the code is only executed if the libary doesn't already exist, or is a newer version
if not AceLibrary then error(MAJOR_VERSION .. " requires AceLibrary.") end
if not AceLibrary:IsNewVersion(MAJOR_VERSION, MINOR_VERSION) then return end

local table_setn
do
	local version = GetBuildInfo()
	if string.find(version, "^2%.") then
		-- 2.0.0
		table_setn = function() end
	else
		table_setn = table.setn
	end
end

local AceOO = {
	error = AceLibrary.error,
	argCheck = AceLibrary.argCheck
}

-- @function	getuid
-- @brief		Obtain a unique string identifier for the object in question.
-- @param t		The object to obtain the uid for.
-- @return		The uid string.
local function pad(cap)
	return string.rep('0', 8 - string.len(cap)) .. cap
end
local function getuid(t)
	local mt = getmetatable(t)
	setmetatable(t, nil)
	local str = tostring(t)
	setmetatable(t, mt)
	local _,_,cap = string.find(str, '[^:]*: 0x(.*)$')
	if cap then return pad(cap) end
	_,_,cap = string.find(str, '[^:]*: (.*)$')
	if cap then return pad(cap) end
end

local function getlibrary(o)
	if type(o) == "table" then
		return o
	elseif type(o) == "string" then
		if not AceLibrary:HasInstance(o) then
			AceOO:error("Library %q does not exist.", o)
		end
		return AceLibrary(o)
	end
end

-- @function		Factory
-- @brief			Construct a factory for the creation of objects.
-- @param obj		The object whose init method will be called on the new factory
--					object.
-- @param newobj	The object whose init method will be called on the new
--					objects that the Factory creates, to initialize them.
-- @param (a1..a20) Arguments which will be passed to obj.init() in addition
--					to the Factory object.
-- @return			The new factory which creates a newobj when its new method is called,
--					or when it is called directly (__call metamethod).
local Factory
do
	local function new(obj, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12,
							a13, a14, a15, a16, a17, a18, a19, a20)
		local t = {}
		local uid = getuid(t)
		local l = getlibrary
		obj:init(t, l(a1), l(a2), l(a3), l(a4), l(a5), l(a6), l(a7),
					l(a8), l(a9), l(a10), l(a11), l(a12), l(a13),
					l(a14), l(a15), l(a16), l(a17), l(a18), l(a19),
					l(a20))
		t.uid = uid
		return t
	end
	
	local function createnew(self, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10,
									a11, a12, a13, a14, a15, a16, a17, a18,
									a19, a20)
		local o = self.prototype
		local l = getlibrary
		return new(o, l(a1), l(a2), l(a3), l(a4), l(a5), l(a6), l(a7),
						l(a8), l(a9), l(a10), l(a11), l(a12), l(a13),
						l(a14), l(a15), l(a16), l(a17), l(a18), l(a19),
						l(a20))
	end

	function Factory(obj, newobj, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10,
									a11, a12, a13, a14, a15, a16, a17, a18,
									a19, a20)
		local t = new(obj, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12,
							a13, a14, a15, a16, a17, a18, a19, a20)
		t.prototype = newobj
		t.new = createnew
		getmetatable(t).__call = t.new
		return t
	end
end


local function objtostring(self)
	if self.ToString then
		return self:ToString()
	elseif self.GetLibraryVersion then
		return (self:GetLibraryVersion())
	elseif self.super then
		local s = "Sub-" .. tostring(self.super)
		local first = true
		if self.interfaces then
			for interface in pairs(self.interfaces) do
				if first then
					s = s .. "(" .. tostring(interface)
					first = false
				else
					s = s .. ", " .. tostring(interface)
				end
			end
		end
		if self.mixins then
			for mixin in pairs(self.mixins) do
				if first then
					s = s .. tostring(mixin)
					first = false
				else
					s = s .. ", " .. tostring(mixin)
				end
			end
		end
		if first then
			if self.uid then
				return s .. ":" .. self.uid
			else
				return s
			end
		else
			return s .. ")"
		end
	else
		return self.uid and 'Subclass:' .. self.uid or 'Subclass'
	end
end

-- @table			Object
-- @brief			Base of all objects, including Class.
--
-- @method			init
-- @brief			Initialize a new object.
-- @param newobject The object to initialize
-- @param class		The class to make newobject inherit from
local Object
do
	Object = {}
	function Object:init(newobject, class)
		local parent = class or self
		if not rawget(newobject, 'uid') then
			newobject.uid = getuid(newobject)
		end
		local mt = {
			__index = parent,
			__tostring = objtostring,
		}
		setmetatable(newobject, mt)
	end
	Object.uid = getuid(Object)
	setmetatable(Object, { __tostring = function() return 'Object' end })
end

local Interface

local function validateInterface(object, interface)
	if not object.class and object.prototype then
		object = object.prototype
	end
	for k,v in pairs(interface.interface) do
		if tostring(type(object[k])) ~= v then
			return false
		end
	end
	if interface.superinterfaces then
		for superinterface in pairs(interface.superinterfaces) do
			if not validateInterface(object, superinterface) then
				return false
			end
		end
	end
	if type(object.class) == "table" and rawequal(object.class.prototype, object) then
		if not object.class.interfaces then
			rawset(object.class, 'interfaces', {})
		end
		object.class.interfaces[interface] = true
	elseif type(object.class) == "table" and type(object.class.prototype) == "table" then
		validateInterface(object.class.prototype, interface)
		-- check if class is proper, thus preventing future checks.
	end
	return true
end

-- @function		inherits
-- @brief			Return whether an Object or Class inherits from a given
--					parent.
-- @param object	Object or Class to check
-- @param parent	Parent to test inheritance from
-- @return			whether an Object or Class inherits from a given
--					parent.
local function inherits(object, parent)
	object = getlibrary(object)
	if type(parent) == "string" then
		if not AceLibrary:HasInstance(parent) then
			return false
		else
			parent = AceLibrary(parent)
		end
	end
	AceOO:argCheck(parent, 2, "table")
	if type(object) ~= "table" then
		return false
	end
	local current
	if object.class then
		current = object.class
	else
		current = object
	end
	if type(current) ~= "table" then
		return false
	end
	if rawequal(current, parent) then
		return true
	end
	if parent.class then
		while true do
			if rawequal(current, Object) then
				break
			end
			if current.mixins then
				for mixin in pairs(current.mixins) do
					if rawequal(mixin, parent) then
						return true
					end
				end
			end
			if current.interfaces then
				for interface in pairs(current.interfaces) do
					if rawequal(interface, parent) then
						return true
					end
				end
			end
			current = current.super
			if type(current) ~= "table" then
				break
			end
		end
		
		local isInterface = false
		local curr = parent.class
		while true do
			if rawequal(curr, Object) then
				break
			elseif rawequal(curr, Interface) then
				isInterface = true
				break
			end
			curr = curr.super
			if type(curr) ~= "table" then
				break
			end
		end
		return isInterface and validateInterface(object, parent)
	else
		while true do
			if rawequal(current, parent) then
				return true
			elseif rawequal(current, Object) then
				return false
			end
			current = current.super
			if type(current) ~= "table" then
				return false
			end
		end
	end
end

-- @table			Class
-- @brief			An object factory which sets up inheritence and supports
--					'mixins'.
--
-- @metamethod		Class call
-- @brief			Call ClassFactory:new() to create a new class.
--
-- @method			Class new
-- @brief			Construct a new object.
-- @param (a1..a20) Arguments to pass to the object init function.
-- @return			The new object.
--
-- @method			Class init
-- @brief			Initialize a new class.
-- @param parent	Superclass.
-- @param (a1..a20) Mixins.
--
-- @method			Class ToString
-- @return			A string representing the object, in this case 'Class'.
local initStatus
local Class
local Mixin
local autoEmbed = false
local function traverseInterfaces(bit, total)
	if bit.superinterfaces then
		for interface in pairs(bit.superinterfaces) do
			if not total[interface] then
				total[interface] = true
				traverseInterfaces(interface, total)
			end
		end
	end
end
local class_new
do
	Class = Factory(Object, setmetatable({}, {__index = Object}), Object)
	Class.super = Object
	
	local function protostring(t)
		return '<' .. tostring(t.class) .. ' prototype>'
	end
	local function classobjectstring(t)
		if t.ToString then
			return t:ToString()
		elseif t.GetLibraryVersion then
			return (t:GetLibraryVersion())
		else
			return '<' .. tostring(t.class) .. ' instance>'
		end
	end
	local function classobjectequal(self, other)
		if type(self) == "table" and self.Equals then
			return self:Equals(other)
		elseif type(other) == "table" and other.Equals then
			return other:Equals(self)
		elseif type(self) == "table" and self.CompareTo then
			return self:CompareTo(other) == 0
		elseif type(other) == "table" and other.CompareTo then
			return other:CompareTo(self) == 0
		else
			return rawequal(self, other)
		end
	end
	local function classobjectlessthan(self, other)
		if type(self) == "table" and self.IsLessThan then
			return self:IsLessThan(other)
		elseif type(other) == "table" and other.IsLessThanOrEqualTo then
			return not other:IsLessThanOrEqualTo(self)
		elseif type(self) == "table" and self.CompareTo then
			return self:CompareTo(other) < 0
		elseif type(other) == "table" and other.CompareTo then
			return other:CompareTo(self) > 0
		elseif type(other) == "table" and other.IsLessThan and other.Equals then
			return other:Equals(self) or other:IsLessThan(self)
		else
			AceOO:error("cannot compare two objects")
		end
	end
	local function classobjectlessthanequal(self, other)
		if type(self) == "table" and self.IsLessThanOrEqualTo then
			return self:IsLessThanOrEqualTo(other)
		elseif type(other) == "table" and other.IsLessThan then
			return not other:IsLessThan(self)
		elseif type(self) == "table" and self.CompareTo then
			return self:CompareTo(other) <= 0
		elseif type(other) == "table" and other.CompareTo then
			return other:CompareTo(self) >= 0
		elseif type(self) == "table" and self.IsLessThan and self.Equals then
			return self:Equals(other) or self:IsLessThan(other)
		else
			AceOO:error("cannot compare two incompatible objects")
		end
	end
	local function classobjectadd(self, other)
		if type(self) == "table" and self.Add then
			return self:Add(other)
		else
			AceOO:error("cannot add two incompatible objects")
		end
	end
	local function classobjectsub(self, other)
		if type(self) == "table" and self.Subtract then
			return self:Subtract(other)
		else
			AceOO:error("cannot subtract two incompatible objects")
		end
	end
	local function classobjectunm(self, other)
		if type(self) == "table" and self.UnaryNegation then
			return self:UnaryNegation(other)
		else
			AceOO:error("attempt to negate an incompatible object")
		end
	end
	local function classobjectmul(self, other)
		if type(self) == "table" and self.Multiply then
			return self:Multiply(other)
		else
			AceOO:error("cannot multiply two incompatible objects")
		end
	end
	local function classobjectdiv(self, other)
		if type(self) == "table" and self.Divide then
			return self:Divide(other)
		else
			AceOO:error("cannot divide two incompatible objects")
		end
	end
	local function classobjectpow(self, other)
		if type(self) == "table" and self.Exponent then
			return self:Exponent(other)
		else
			AceOO:error("cannot exponentiate two incompatible objects")
		end
	end
	local function classobjectconcat(self, other)
		if type(self) == "table" and self.Concatenate then
			return self:Concatenate(other)
		else
			AceOO:error("cannot concatenate two incompatible objects")
		end
	end
	function class_new(self, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12,
						a13, a14, a15, a16, a17, a18, a19, a20)
		if self.virtual then
			AceOO:error("Cannot instantiate a virtual class.")
		end
		
		local o = self.prototype
		local newobj = {}
		if o.class and o.class.instancemeta then
			setmetatable(newobj, o.class.instancemeta)
		else
			Object:init(newobj, o)
		end
		
		if self.interfaces and not self.interfacesVerified then
			-- Verify the interfaces
			
			for interface in pairs(self.interfaces) do
				for field,kind in pairs(interface.interface) do
					if tostring(type(newobj[field])) ~= kind then
						AceOO:error("Class did not satisfy all interfaces. %q is required to be a %s. It is a %s", field, kind, tostring(type(newobj[field])))
					end
				end
			end
			self.interfacesVerified = true
		end
		local tmp = initStatus
		initStatus = newobj
		newobj:init(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12,
						a13, a14, a15, a16, a17, a18, a19, a20)
		if initStatus then
			initStatus = tmp
			AceOO:error("Initialization not completed, be sure to call the superclass's init method.")
			return
		end
		initStatus = tmp
		return newobj
	end
	local classmeta = {
		__tostring = objtostring,
		__call = function(self, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12,
						a13, a14, a15, a16, a17, a18, a19, a20)
			return self:new(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12,
						a13, a14, a15, a16, a17, a18, a19, a20)
		end,
	}
	function Class:init(newclass, parent, a1, a2, a3, a4, a5, a6, a7, a8, a9,
											a10, a11, a12, a13, a14, a15, a16,
											a17, a18, a19, a20)
		parent = parent or self
		
		local total
		
		if parent.class then
			total = {
				parent, a1, a2, a3, a4, a5, a6, a7, a8, a9,
						a10, a11, a12, a13, a14, a15, a16,
						a17, a18, a19, a20
			}
			parent = self
		else
			total = {
				a1, a2, a3, a4, a5, a6, a7, a8, a9,
				a10, a11, a12, a13, a14, a15, a16,
				a17, a18, a19, a20
			}
		end
		if not inherits(parent, Class) then
			AceOO:error("Classes must inherit from a proper class")
		end
		if parent.sealed then
			AceOO:error("Cannot inherit from a sealed class")
		end
		for i,v in ipairs(total) do
			if inherits(v, Mixin) and v.class then
				if not newclass.mixins then
					newclass.mixins = {}
				end
				if newclass.mixins[v] then
					AceOO:error("Cannot explicitly inherit from the same mixin twice")
				end
				newclass.mixins[v] = true
			elseif inherits(v, Interface) and v.class then
				if not newclass.interfaces then
					newclass.interfaces = {}
				end
				if newclass.interfaces[v] then
					AceOO:error("Cannot explicitly inherit from the same interface twice")
				end
				newclass.interfaces[v] = true
			else
				AceOO:error("Classes can only inherit from one or zero classes and any number of mixins or interfaces")
			end
		end
		if parent.interfaces then
			if newclass.interfaces then
				for interface in pairs(parent.interfaces) do
					newclass.interfaces[interface] = true
				end
			else
				newclass.interfaces = parent.interfaces
			end
		end
		for k in pairs(total) do
			total[k] = nil
		end
		table_setn(total, 0)
		
		newclass.super = parent
		
		newclass.prototype = setmetatable(total, {
			__index = parent.prototype,
			__tostring = protostring,
		})
		total = nil
		
		newclass.instancemeta = {
			__index = newclass.prototype,
			__tostring = classobjectstring,
			__eq = classobjectequal,
			__lt = classobjectlessthan,
			__le = classobjectlessthanequal,
			__add = classobjectadd,
			__sub = classobjectsub,
			__unm = classobjectunm,
			__mul = classobjectmul,
			__div = classobjectdiv,
			__pow = classobjectpow,
			__concat = classobjectconcat,
		}
		
		setmetatable(newclass, classmeta)
		
		newclass.new = class_new
		
		if newclass.mixins then
			-- Fold in the mixins
			local err, msg
			for mixin in pairs(newclass.mixins) do
				local ret
				autoEmbed = true
				ret, msg = pcall(mixin.embed, mixin, newclass.prototype)
				autoEmbed = false
				if not ret then
					err = true
					break
				end
			end
	
			if err then
				local pt = newclass.prototype
				for k,v in pairs(pt) do
					pt[k] = nil
				end
	
				-- method conflict
				AceOO:error(msg)
			end
		end
		
		newclass.prototype.class = newclass
		
		if newclass.interfaces then
			for interface in pairs(newclass.interfaces) do
				traverseInterfaces(interface, newclass.interfaces)
			end
		end
		if newclass.mixins then
			for mixin in pairs(newclass.mixins) do
				if mixin.interfaces then
					if not newclass.interfaces then
						newclass.interfaces = {}
					end
					for interface in pairs(mixin.interfaces) do
						newclass.interfaces[interface] = true
					end
				end
			end
		end
	end
	function Class:ToString()
		if type(self.GetLibraryVersion) == "function" then
			return (self:GetLibraryVersion())
		else
			return "Class"
		end
	end
	
	local tmp
	function Class.prototype:init()
		if rawequal(self, initStatus) then
			initStatus = nil
		else
			AceOO:error("Improper self passed to init. You must do MyClass.super.prototype.init(self, ...)", 2)
		end
		self.uid = getuid(self)
		local current = self.class
		while true do
			if current == Class then
				break
			end
			if current.mixins then
				for mixin in pairs(current.mixins) do
					if type(mixin.OnInstanceInit) == "function" then
						mixin:OnInstanceInit(self)
					end
				end
			end
			current = current.super
		end
	end
end


-- @object	ClassFactory
-- @brief	A factory for creating classes.	Rarely used directly.
local ClassFactory = Factory(Object, Class, Object)

function Class:new(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11,
					a12, a13, a14, a15, a16, a17, a18, a19, a20)
	local x = ClassFactory:new(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11,
							a12, a13, a14, a15, a16, a17, a18, a19, a20)
	if AceOO.classes then
		AceOO.classes[x] = true
	end
	return x
end
getmetatable(Class).__call = Class.new

-- @class	Mixin
-- @brief	A class to create mixin objects, which contain methods that get
-- "mixed in" to class prototypes.
--
-- @object	Mixin prototype
-- @brief	The prototype that mixin objects inherit their methods from.
--
-- @method	Mixin prototype embed
-- @brief	Mix in the methods of our object which are listed in our interface
--		 to the supplied target table.
--
-- @method	Mixin prototype init
-- @brief	Initialize the mixin object.
-- @param	newobj	 The new object we're initializing.
-- @param	interface	The interface we implement (the list of methods our
--					prototype provides which should be mixed into the target
--					table by embed).
do
	Mixin = Class()
	function Mixin:ToString()
		if self.GetLibraryVersion then
			return (self:GetLibraryVersion())
		else
			return 'Mixin'
		end
	end
	local function _Embed(state, field, target)
		field = next(state.export, field)
		if field == nil then
			return
		end

		if rawget(target, field) or (target[field] and target[field] ~= state[field]) then
			AceOO:error("Method conflict in attempt to mixin. Field %q", field)
		end

		target[field] = state[field]

		local ret,msg = pcall(_Embed, state, field, target)
		if not ret then
			-- Mix in the next method according to the defined interface.	If that
			-- fails due to a conflict, re-raise to back out the previous mixed
			-- methods.

			target[field] = nil
			AceOO:error(msg)
		end
	end
	function Mixin.prototype:embed(target)
		local mt = getmetatable(target)
		setmetatable(target, nil)
		local err, msg = pcall(_Embed, self, nil, target)
		if not err then
			setmetatable(target, mt)
			AceOO:error(msg)
			return
		end
		if type(self.embedList) == "table" then
			self.embedList[target] = true
		end
		if type(target.class) ~= "table" then
			target[self] = true
		end
		if not autoEmbed and type(self.OnManualEmbed) == "function" then
			self:OnManualEmbed(target)
		end
		setmetatable(target, mt)
	end
	
	function Mixin.prototype:activate(oldLib, oldDeactivate)
		if oldLib and oldLib.embedList then
			for target in pairs(oldLib.embedList) do
				local mt = getmetatable(target)
				setmetatable(target, nil)
				for field in pairs(oldLib.export) do
					target[field] = nil
				end
				setmetatable(target, mt)
			end
			self.embedList = oldLib.embedList
			for target in pairs(self.embedList) do
				self:embed(target)
			end
		else
			self.embedList = setmetatable({}, {__mode="k"})
		end
	end
	
	function Mixin.prototype:init(export, a1, a2, a3, a4, a5, a6, a7, a8, a9,
											a10, a11, a12, a13, a14, a15, a16,
											a17, a18, a19, a20)
		AceOO:argCheck(export, 2, "table")
		for k,v in pairs(export) do
			if type(k) ~= "number" then
				AceOO:error("All keys to argument #2 must be numbers.")
			elseif type(v) ~= "string" then
				AceOO:error("All values to argument #2 must be strings.")
			end
		end
		local num = table.getn(export)
		for i = 1, num do
			local v = export[i]
			export[i] = nil
			export[v] = true
		end
		table_setn(export, 0)
		local interfaces
		if a1 then
			local l = getlibrary
			interfaces = { l(a1), l(a2), l(a3), l(a4), l(a5), l(a6), l(a7), l(a8),
				l(a9), l(a10), l(a11), l(a12), l(a13), l(a14), l(a15), l(a16),
				l(a17), l(a18), l(a19), l(a20) }
			for _,v in ipairs(interfaces) do
				if not v.class or not inherits(v, Interface) then
					AceOO:error("Mixins can inherit only from interfaces")
				end
			end
			local num = table.getn(interfaces)
			for i = 1, num do
				local v = interfaces[i]
				interfaces[i] = nil
				interfaces[v] = true
			end
			table_setn(interfaces, 0)
			for interface in pairs(interfaces) do
				traverseInterfaces(interface, interfaces)
			end
			for interface in pairs(interfaces) do
				for field,kind in pairs(interface.interface) do
					if kind ~= "nil" then
						local good = false
						for bit in pairs(export) do
							if bit == field then
								good = true
								break
							end
						end
						if not good then
							AceOO:error("Mixin does not fully accommodate field %q", field)
						end
					end
				end
			end
		end
		self.super = Mixin.prototype
		Mixin.super.prototype.init(self)
		self.export = export
		self.interfaces = interfaces
	end
end

-- @class Interface
-- @brief A class to create interfaces, which contain contracts that classes
--			which inherit from this must comply with.
--
-- @object Interface prototype
-- @brief	The prototype that interface objects must adhere to.
--
-- @method Interface prototype init
-- @brief	Initialize the mixin object.
-- @param	interface	The interface we contract (the hash of fields forced).
-- @param	(a1..a20)	Superinterfaces
do
	Interface = Class()
	function Interface:ToString()
		if self.GetLibraryVersion then
			return (self:GetLibraryVersion())
		else
			return 'Instance'
		end
	end
	function Interface.prototype:init(interface, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20)
		Interface.super.prototype.init(self)
		AceOO:argCheck(interface, 2, "table")
		for k,v in pairs(interface) do
			if type(k) ~= "string" then
				AceOO:error("All keys to argument #2 must be numbers.")
			elseif type(v) ~= "string" then
				AceOO:error("All values to argument #2 must be strings.")
			elseif v ~= "nil" and v ~= "string" and v ~= "number" and v ~= "table" and v ~= "function" then
				AceOO:error('All values to argument #2 must either be "nil", "string", "number", "table", or "function".')
			end
		end
		local l = getlibrary
		a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20 = l(a1), l(a2), l(a3), l(a4), l(a5), l(a6), l(a7), l(a8), l(a9), l(a10), l(a11), l(a12), l(a13), l(a14), l(a15), l(a16), l(a17), l(a18), l(a19), l(a20)
		if a1 then
			self.superinterfaces = {a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20}
			for k,v in ipairs(self.superinterfaces) do
				if not inherits(v, Interface) or not v.class then
					AceOO:error('Cannot provide a non-Interface to inherit from')
				end
			end
			local num = table.getn(self.superinterfaces)
			for i = 1, num do
				local v = self.superinterfaces[i]
				self.superinterfaces[i] = nil
				self.superinterfaces[v] = true
			end
			table_setn(self.superinterfaces, 0)
		end
		self.interface = interface
	end
end

-- @function Classpool
-- @brief	Obtain a read only class from our pool of classes, indexed by the
--		 superclass and mixins.
-- @param	sc		 The superclass of the class we want.
-- @param	(m1..m20)	Mixins of the class we want's objects.
-- @return A read only class from the class pool.
local Classpool
do
	local pool = setmetatable({}, {__mode = 'v'})
	local function newindex(k, v)
		AceOO:error('Attempt to modify a read-only class.')
	end
	local function protonewindex(k, v)
		AceOO:error('Attempt to modify a read-only class prototype.')
	end
	local function ts(bit)
		if type(bit) ~= "table" then
			return tostring(bit)
		elseif getmetatable(bit) and bit.__tostring then
			return tostring(bit)
		elseif type(bit.GetLibraryVersion) == "function" then
			return bit:GetLibraryVersion()
		else
			return tostring(bit)
		end
	end
	local t
	local function getcomplexuid(sc, m1, m2, m3, m4, m5, m6, m7, m8, m9,
		m10, m11, m12, m13, m14, m15, m16, m17, m18, m19, m20)
		if not t then t = {} end
		if sc then if sc.uid then table.insert(t, sc.uid) else AceOO:error("%s is not an appropriate class/mixin", ts(sc)) end
		if m1 then if m1.uid then table.insert(t, m1.uid) else AceOO:error("%s is not an appropriate mixin", ts(m1)) end
		if m2 then if m2.uid then table.insert(t, m2.uid) else AceOO:error("%s is not an appropriate mixin", ts(m2)) end
		if m3 then if m3.uid then table.insert(t, m3.uid) else AceOO:error("%s is not an appropriate mixin", ts(m3)) end
		if m4 then if m4.uid then table.insert(t, m4.uid) else AceOO:error("%s is not an appropriate mixin", ts(m4)) end
		if m5 then if m5.uid then table.insert(t, m5.uid) else AceOO:error("%s is not an appropriate mixin", ts(m5)) end
		if m6 then if m6.uid then table.insert(t, m6.uid) else AceOO:error("%s is not an appropriate mixin", ts(m6)) end
		if m7 then if m7.uid then table.insert(t, m7.uid) else AceOO:error("%s is not an appropriate mixin", ts(m7)) end
		if m8 then if m8.uid then table.insert(t, m8.uid) else AceOO:error("%s is not an appropriate mixin", ts(m8)) end
		if m9 then if m9.uid then table.insert(t, m9.uid) else AceOO:error("%s is not an appropriate mixin", ts(m9)) end
		if m10 then if m10.uid then table.insert(t, m10.uid) else AceOO:error("%s is not an appropriate mixin", ts(m10)) end
		if m11 then if m11.uid then table.insert(t, m11.uid) else AceOO:error("%s is not an appropriate mixin", ts(m11)) end
		if m12 then if m12.uid then table.insert(t, m12.uid) else AceOO:error("%s is not an appropriate mixin", ts(m12)) end
		if m13 then if m13.uid then table.insert(t, m13.uid) else AceOO:error("%s is not an appropriate mixin", ts(m13)) end
		if m14 then if m14.uid then table.insert(t, m14.uid) else AceOO:error("%s is not an appropriate mixin", ts(m14)) end
		if m15 then if m15.uid then table.insert(t, m15.uid) else AceOO:error("%s is not an appropriate mixin", ts(m15)) end
		if m16 then if m16.uid then table.insert(t, m16.uid) else AceOO:error("%s is not an appropriate mixin", ts(m16)) end
		if m17 then if m17.uid then table.insert(t, m17.uid) else AceOO:error("%s is not an appropriate mixin", ts(m17)) end
		if m18 then if m18.uid then table.insert(t, m18.uid) else AceOO:error("%s is not an appropriate mixin", ts(m18)) end
		if m19 then if m19.uid then table.insert(t, m19.uid) else AceOO:error("%s is not an appropriate mixin", ts(m19)) end
		if m20 then if m20.uid then table.insert(t, m20.uid) else AceOO:error("%s is not an appropriate mixin", ts(m20)) end
		end end end end end end end end end end end end end end end end end end end end end
		table.sort(t)
		local uid = table.concat(t, '')
		for k in pairs(t) do t[k] = nil end
		table_setn(t, 0)
		return uid
	end
	local classmeta
	function Classpool(sc, m1, m2, m3, m4, m5, m6, m7, m8, m9,
		m10, m11, m12, m13, m14, m15, m16,
		m17, m18, m19, m20)
		local l = getlibrary
		sc, m1, m2, m3, m4, m5, m6, m7, m8, m9, m10, m11, m12, m13, m14, m15, m16, m17, m18, m19, m20 = l(sc), l(m1), l(m2), l(m3), l(m4), l(m5), l(m6), l(m7), l(m8), l(m9), l(m10), l(m11), l(m12), l(m13), l(m14), l(m15), l(m16), l(m17), l(m18), l(m19), l(m20)
		if sc and sc.class then
			sc, m1, m2, m3, m4, m5, m6, m7, m8, m9, m10, m11, m12, m13, m14, m15, m16, m17, m18, m19, m20 = Class, sc, m1, m2, m3, m4, m5, m6, m7, m8, m9, m10, m11, m12, m13, m14, m15, m16, m17, m18, m19
		end
		sc = sc or Class
		local key = getcomplexuid(sc, m1, m2, m3, m4, m5, m6, m7, m8, m9, m10, m11, m12, m13, m14, m15, m16, m17, m18, m19, m20)
		if not pool[key] then
			local class = Class(sc, m1, m2, m3, m4, m5, m6, m7, m8, m9,
			m10, m11, m12, m13, m14, m15, m16, m17,
			m18, m19, m20)
			if not classmeta then
				classmeta = {}
				local mt = getmetatable(class)
				for k,v in pairs(mt) do
					classmeta[k] = v
				end
				classmeta.__newindex = newindex
			end
			-- Prevent the user from adding methods to this class.
			-- NOTE: I'm not preventing modifications of existing class members,
			-- but it's likely that only a truly malicious user will be doing so.
			class.sealed = true
			setmetatable(class, classmeta)
			getmetatable(class.prototype).__newindex = protonewindex
			pool[key] = class
		end
		return pool[key]
	end
end

AceOO.Factory = Factory
AceOO.Object = Object
AceOO.Class = Class
AceOO.Mixin = Mixin
AceOO.Interface = Interface
AceOO.Classpool = Classpool
AceOO.inherits = inherits

-- Library handling bits

local function activate(self, oldLib, oldDeactivate)
	AceOO = self
	Factory = self.Factory
	Object = self.Object
	Class = self.Class
	ClassFactory.prototype = Class
	Mixin = self.Mixin
	Interface = self.Interface
	Classpool = self.Classpool
	
	if oldLib then
		self.classes = oldLib.classes
	end
	if not self.classes then
		self.classes = setmetatable({}, {__mode="k"})
	else
		for class in pairs(self.classes) do
			class.new = class_new
		end
	end
	
	if oldDeactivate then
		oldDeactivate(oldLib)
	end
end

AceLibrary:Register(AceOO, MAJOR_VERSION, MINOR_VERSION, activate)
AceOO = AceLibrary(MAJOR_VERSION)
```

## lib/Babble-Class-2.2/Babble-Class-2.2.lua
```lua
--[[
Name: Babble-Class-2.2
Revision: $Rev: 14410 $
Author(s): ckknight (ckknight@gmail.com)
Website: http://ckknight.wowinterface.com/
Documentation: http://wiki.wowace.com/index.php/Babble-Class-2.2
SVN: http://svn.wowace.com/root/trunk/Babble-2.2/Babble-Class-2.2
Description: A library to provide localizations for classes.
Dependencies: AceLibrary, AceLocale-2.2
]]

local MAJOR_VERSION = "Babble-Class-2.2"
local MINOR_VERSION = tonumber(string.sub("$Revision: 14410 $", 12, -3))

if not AceLibrary then error(MAJOR_VERSION .. " requires AceLibrary") end

if not AceLibrary:HasInstance("AceLocale-2.2") then error(MAJOR_VERSION .. " requires AceLocale-2.2") end

local _, x = AceLibrary("AceLocale-2.2"):GetLibraryVersion()
MINOR_VERSION = MINOR_VERSION * 100000 + x

if not AceLibrary:IsNewVersion(MAJOR_VERSION, MINOR_VERSION) then return end

local BabbleClass = AceLibrary("AceLocale-2.2"):new(MAJOR_VERSION)

-- uncomment below for debug information
-- BabbleClass:EnableDebugging()

BabbleClass:RegisterTranslations("enUS", function()
	return {
		["Warlock"] = true,
		["Warrior"] = true,
		["Hunter"] = true,
		["Mage"] = true,
		["Priest"] = true,
		["Druid"] = true,
		["Paladin"] = true,
		["Shaman"] = true,
		["Rogue"] = true,
	}
end)

BabbleClass:RegisterTranslations("deDE", function()
	return {
		["Warlock"] = "Hexenmeister",
		["Warrior"] = "Krieger",
		["Hunter"] = "J�ger",
		["Mage"] = "Magier",
		["Priest"] = "Priester",
		["Druid"] = "Druide",
		["Paladin"] = "Paladin",
		["Shaman"] = "Schamane",
		["Rogue"] = "Schurke",
	}
end)

BabbleClass:RegisterTranslations("frFR", function()
	return {
		["Warlock"] = "D\195\169moniste",
		["Warrior"] = "Guerrier",
		["Hunter"] = "Chasseur",
		["Mage"] = "Mage",
		["Priest"] = "Pr\195\170tre",
		["Druid"] = "Druide",
		["Paladin"] = "Paladin",
		["Shaman"] = "Chaman",
		["Rogue"] = "Voleur",
	}
end)

BabbleClass:RegisterTranslations("zhCN", function()
	return {
		["Warlock"] = "??",
		["Warrior"] = "??",
		["Hunter"] = "??",
		["Mage"] = "??",
		["Priest"] = "??",
		["Druid"] = "???",
		["Paladin"] = "???",
		["Shaman"] = "????",
		["Rogue"] = "??",
	}
end)

BabbleClass:RegisterTranslations("zhTW", function()
	return {
		["Warlock"] = "??",
		["Warrior"] = "??",
		["Hunter"] = "??",
		["Mage"] = "??",
		["Priest"] = "??",
		["Druid"] = "???",
		["Paladin"] = "???",
		["Shaman"] = "??",
		["Rogue"] = "??",
	}
end)

BabbleClass:RegisterTranslations("koKR", function()
	return {
		["Warlock"] = "????",
		["Warrior"] = "??",
		["Hunter"] = "???",
		["Mage"] = "???",
		["Priest"] = "??",
		["Druid"] = "????",
		["Paladin"] = "???",
		["Shaman"] = "???",
		["Rogue"] = "??",
	}
end)

BabbleClass:Debug()
BabbleClass:SetStrictness(true)

function BabbleClass:GetColor(class)
	self:argCheck(class, 2, "string")
	if self:HasReverseTranslation(class) then
		class = string.upper(self:GetReverseTranslation(class))
	else
		class = string.upper(class)
	end
	if RAID_CLASS_COLORS and RAID_CLASS_COLORS[class] then
		return RAID_CLASS_COLORS[class].r, RAID_CLASS_COLORS[class].g, RAID_CLASS_COLORS[class].b
	end
	return 0.63, 0.63, 0.63
end

function BabbleClass:GetHexColor(class)
	self:argCheck(class, 2, "string")
	local r, g, b = self:GetColor(class)
	return string.format("%02x%02x%02x", r * 255, g * 255, b * 255)
end

AceLibrary:Register(BabbleClass, MAJOR_VERSION, MINOR_VERSION)
BabbleClass = nil
```

## lib/Deformat-2.0/Deformat-2.0.lua
```lua
--[[
Name: Deformat-2.0
Revision: $Rev: 6804 $
Author(s): ckknight (ckknight@gmail.com)
Website: http://ckknight.wowinterface.com/
Documentation: http://wiki.wowace.com/index.php/Deformat-2.0
SVN: http://svn.wowace.com/root/trunk/Deformat/Deformat-2.0
Description: A library to deformat format strings.
Dependencies: AceLibrary
]]

local MAJOR_VERSION = "Deformat-2.0"
local MINOR_VERSION = "$Revision: 6804 $"


if not AceLibrary then error(MAJOR_VERSION .. " requires AceLibrary") end
if not AceLibrary:IsNewVersion(MAJOR_VERSION, MINOR_VERSION) then return end

local Deformat = {}

do
	local sequences = {
		["%d*d"] = "%%-?%%d+",
		["s"] = ".+",
		["[fg]"] = "%%-?%%d+%%.%%d+",
		["%%%.%d[fg]"] = "%%-?%%d+%%.?%%d*",
		["c"] = ".",
	}
	local curries = {}
	
	local function doNothing(item)
		return item
	end
	local v = {}
	
	local function concat(a1, a2, a3, a4, a5)
		local left, right
		if not a2 then
			return a1
		elseif not a3 then
			left, right = a1, a2
		elseif not a4 then
			return concat(concat(a1, a2), a3)
		elseif not a5 then
			return concat(concat(concat(a1, a2), a3), a4)
		else
			return concat(concat(concat(concat(a1, a2), a3), a4), a5)
		end
		if not string.find(left, "%%1%$") and not string.find(right, "%%1%$") then
			return left .. right
		elseif not string.find(right, "%%1%$") then
			local i
			for j = 9, 1, -1 do
				if string.find(left, "%%" .. j .. "%$") then
					i = j
					break
				end
			end
			while true do
				local first
				local firstPat
				for x, y in pairs(sequences) do
					local i = string.find(right, "%%" .. x)
					if not first or (i and i < first) then
						first = i
						firstPat = x
					end
				end
				if not first then
					break
				end
				i = i + 1
				right = string.gsub(right, "%%(" .. firstPat .. ")", "%%" .. i .. "$%1")
			end
			return left .. right
		elseif not string.find(left, "%%1%$") then
			local i = 1
			while true do
				local first
				local firstPat
				for x, y in pairs(sequences) do
					local i = string.find(left, "%%" .. x)
					if not first or (i and i < first) then
						first = i
						firstPat = x
					end
				end
				if not first then
					break
				end
				i = i + 1
				left = string.gsub(left, "%%(" .. firstPat .. ")", "%%" .. i .. "$%1")
			end
			return concat(left, right)
		else
			local i
			for j = 9, 1, -1 do
				if string.find(left, "%%" .. j .. "%$") then
					i = j
					break
				end
			end
			local j
			for k = 9, 1, -1 do
				if string.find(right, "%%" .. k .. "%$") then
					j = k
					break
				end
			end
			for k = j, 1, -1 do
				right = string.gsub(right, "%%" .. k .. "%$", "%%" .. k + i .. "%$")
			end
			return left .. right
		end
	end
	
	local function Curry(a1, a2, a3, a4, a5)
		local pattern = concat(a1, a2, a3, a4, a5)
		if not string.find(pattern, "%%1%$") then
			local unpattern = string.gsub(pattern, "([%(%)%.%*%+%-%[%]%?%^%$%%])", "%%%1")
			local f = {}
			local i = 0
			while true do
				local first
				local firstPat
				for x, y in pairs(sequences) do
					local i = string.find(unpattern, "%%%%" .. x)
					if not first or (i and i < first) then
						first = i
						firstPat = x
					end
				end
				if not first then
					break
				end
				unpattern = string.gsub(unpattern, "%%%%" .. firstPat, "(" .. sequences[firstPat] .. ")", 1)
				i = i + 1
				if firstPat == "c" or firstPat == "s" then
					table.insert(f, doNothing)
				else
					table.insert(f, tonumber)
				end
			end
			unpattern = "^" .. unpattern .. "$"
			local _,alpha, bravo, charlie, delta, echo, foxtrot, golf, hotel, india
			if i == 0 then
				return
			elseif i == 1 then
				return function(text)
					_,_,alpha = string.find(text, unpattern)
					if alpha then
						return f[1](alpha)
					end
				end
			elseif i == 2 then
				return function(text)
					_,_,alpha, bravo = string.find(text, unpattern)
					if alpha then
						return f[1](alpha), f[2](bravo)
					end
				end
			elseif i == 3 then
				return function(text)
					_,_,alpha, bravo, charlie = string.find(text, unpattern)
					if alpha then
						return f[1](alpha), f[2](bravo), f[3](charlie)
					end
				end
			elseif i == 4 then
				return function(text)
					_,_,alpha, bravo, charlie, delta = string.find(text, unpattern)
					if alpha then
						return f[1](alpha), f[2](bravo), f[3](charlie), f[4](delta)
					end
				end
			elseif i == 5 then
				return function(text)
					_,_,alpha, bravo, charlie, delta, echo = string.find(text, unpattern)
					if alpha then
						return f[1](alpha), f[2](bravo), f[3](charlie), f[4](delta), f[5](echo)
					end
				end
			elseif i == 6 then
				return function(text)
					_,_,alpha, bravo, charlie, delta, echo, foxtrot = string.find(text, unpattern)
					if alpha then
						return f[1](alpha), f[2](bravo), f[3](charlie), f[4](delta), f[5](echo), f[6](foxtrot)
					end
				end
			elseif i == 7 then
				return function(text)
					_,_,alpha, bravo, charlie, delta, echo, foxtrot, golf = string.find(text, unpattern)
					if alpha then
						return f[1](alpha), f[2](bravo), f[3](charlie), f[4](delta), f[5](echo), f[6](foxtrot), f[7](golf)
					end
				end
			elseif i == 8 then
				return function(text)
					_,_,alpha, bravo, charlie, delta, echo, foxtrot, golf, hotel = string.find(text, unpattern)
					if alpha then
						return f[1](alpha), f[2](bravo), f[3](charlie), f[4](delta), f[5](echo), f[6](foxtrot), f[7](golf), f[8](hotel)
					end
				end
			else
				return function(text)
					_,_,alpha, bravo, charlie, delta, echo, foxtrot, golf, hotel, india = string.find(text, unpattern)
					if alpha then
						return f[1](alpha), f[2](bravo), f[3](charlie), f[4](delta), f[5](echo), f[6](foxtrot), f[7](golf), f[8](hotel), f[9](india)
					end
				end
			end
		else
			local o = {}
			local f = {}
			local unpattern = string.gsub(pattern, "([%(%)%.%*%+%-%[%]%?%^%$%%])", "%%%1")
			local i = 1
			while true do
				local pat
				for x, y in pairs(sequences) do
					if not pat and string.find(unpattern, "%%%%" .. i .. "%%%$" .. x) then
						pat = x
						break
					end
				end
				if not pat then
					break
				end
				unpattern = string.gsub(unpattern, "%%%%" .. i .. "%%%$" .. pat, "(" .. sequences[pat] .. ")", 1)
				if pat == "c" or pat  == "s" then
					table.insert(f, doNothing)
				else
					table.insert(f, tonumber)
				end
				i = i + 1
			end
			i = 1
			string.gsub(pattern, "%%(%d)%$", function(w) o[i] = tonumber(w); i = i + 1; end)
			v[1],v[2],v[3],v[4],v[5],v[6],v[7],v[8],v[9] = nil
			for x, y in pairs(f) do
				v[x] = f[y]
			end
			for x, y in pairs(v) do
				f[x] = v[x]
			end
			unpattern = "^" .. unpattern .. "$"
			i = i - 1
			if i == 0 then
				return function(text)
					return
				end
			elseif i == 1 then
				return function(text)
					_,_,v[1] = string.find(text, unpattern)
					if v[1] then
						return f[1](v[1])
					end
				end
			elseif i == 2 then
				return function(text)
					_,_,v[1],v[2] = string.find(text, unpattern)
					if v[1] then
						return f[1](v[o[1]]), f[2](v[o[2]])
					end
				end
			elseif i == 3 then
				return function(text)
					_,_,v[1],v[2],v[3] = string.find(text, unpattern)
					if v[1] then
						return f[1](v[o[1]]), f[2](v[o[2]]), f[3](v[o[3]])
					end
				end
			elseif i == 4 then
				return function(text)
					_,_,v[1],v[2],v[3],v[4] = string.find(text, unpattern)
					if v[1] then
						return f[1](v[o[1]]), f[2](v[o[2]]), f[3](v[o[3]]), f[4](v[o[4]])
					end
				end
			elseif i == 5 then
				return function(text)
					_,_,v[1],v[2],v[3],v[4],v[5] = string.find(text, unpattern)
					if v[1] then
						return f[1](v[o[1]]), f[2](v[o[2]]), f[3](v[o[3]]), f[4](v[o[4]]), f[5](v[o[5]])
					end
				end
			elseif i == 6 then
				return function(text)
					_,_,v[1],v[2],v[3],v[4],v[5],v[6] = string.find(text, unpattern)
					if v[1] then
						return f[1](v[o[1]]), f[2](v[o[2]]), f[3](v[o[3]]), f[4](v[o[4]]), f[5](v[o[5]]), f[6](v[o[6]])
					end
				end
			elseif i == 7 then
				return function(text)
					_,_,v[1],v[2],v[3],v[4],v[5],v[6],v[7] = string.find(text, unpattern)
					if v[1] then
						return f[1](v[o[1]]), f[2](v[o[2]]), f[3](v[o[3]]), f[4](v[o[4]]), f[5](v[o[5]]), f[6](v[o[6]]), f[7](v[o[7]])
					end
				end
			elseif i == 8 then
				return function(text)
					_,_,v[1],v[2],v[3],v[4],v[5],v[6],v[7],v[8] = string.find(text, unpattern)
					if v[1] then
						return f[1](v[o[1]]), f[2](v[o[2]]), f[3](v[o[3]]), f[4](v[o[4]]), f[5](v[o[5]]), f[6](v[o[6]]), f[7](v[o[7]]), f[8](v[o[8]])
					end
				end
			else
				return function(text)
					_,_,v[1],v[2],v[3],v[4],v[5],v[6],v[7],v[8],v[9] = string.find(text, unpattern)
					if v[1] then
						return f[1](v[o[1]]), f[2](v[o[2]]), f[3](v[o[3]]), f[4](v[o[4]]), f[5](v[o[5]]), f[6](v[o[6]]), f[7](v[o[7]]), f[8](v[o[8]]), f[9](v[o[9]])
					end
				end
			end
		end
	end
	
	function Deformat:Deformat(text, a1, a2, a3, a4, a5)
		self:argCheck(text, 2, "string")
		self:argCheck(a1, 3, "string")
		local pattern = a1
		if a5 then
			pattern = a1 .. a2 .. a3 .. a4 .. a5
		elseif a4 then
			pattern = a1 .. a2 .. a3 .. a4
		elseif a3 then
			pattern = a1 .. a2 .. a3
		elseif a2 then
			pattern = a1 .. a2
		end
		if curries[pattern] == nil then
			curries[pattern] = Curry(a1, a2, a3, a4, a5)
		end
		return curries[pattern](text)
	end
end

local mt = getmetatable(Deformat) or {}
mt.__call = Deformat.Deformat
setmetatable(Deformat, mt)

AceLibrary:Register(Deformat, MAJOR_VERSION, MINOR_VERSION)
Deformat = nil
```

## lib/Dewdrop-2.0/Dewdrop-2.0.lua
```lua
--[[
Name: Dewdrop-2.0
Revision: $Rev: 15465 $
Author(s): ckknight (ckknight@gmail.com)
Website: http://ckknight.wowinterface.com/
Documentation: http://wiki.wowace.com/index.php/Dewdrop-2.0
SVN: http://svn.wowace.com/root/trunk/DewdropLib/Dewdrop-2.0
Description: A library to provide a clean dropdown menu interface.
Dependencies: AceLibrary
]]

local MAJOR_VERSION = "Dewdrop-2.0"
local MINOR_VERSION = "$Revision: 15465 $"

if not AceLibrary then error(MAJOR_VERSION .. " requires AceLibrary") end
if not AceLibrary:IsNewVersion(MAJOR_VERSION, MINOR_VERSION) then return end

local Dewdrop = {}

local lua51 = loadstring("return function(...) return ... end") and true or false

local table_setn = lua51 and function() end or table.setn

local function new(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20)
	local t = {}
	if k1 then t[k1] = v1
	if k2 then t[k2] = v2
	if k3 then t[k3] = v3
	if k4 then t[k4] = v4
	if k5 then t[k5] = v5
	if k6 then t[k6] = v6
	if k7 then t[k7] = v7
	if k8 then t[k8] = v8
	if k9 then t[k9] = v9
	if k10 then t[k10] = v10
	if k11 then t[k11] = v11
	if k12 then t[k12] = v12
	if k13 then t[k13] = v13
	if k14 then t[k14] = v14
	if k15 then t[k15] = v15
	if k16 then t[k16] = v16
	if k17 then t[k17] = v17
	if k18 then t[k18] = v18
	if k19 then t[k19] = v19
	if k20 then t[k20] = v20
	end end end end end end end end end end end end end end end end end end end end
	return t
end
if lua51 then
	new = loadstring("return function(...) local t = {}; for i = 1, select('#', ...), 2 do if select(i, ...) then t[select(i, ...)] = select(i+1, ...); else break; end; end; return t; end")()
end

local tmp
do
	local t = {}
	function tmp(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20)
		for k in pairs(t) do
			t[k] = nil
		end
		if type(k1) == "table" then
			for k,v in pairs(k1) do
				t[k] = v
			end
		else
			if k1 then t[k1] = v1
			if k2 then t[k2] = v2
			if k3 then t[k3] = v3
			if k4 then t[k4] = v4
			if k5 then t[k5] = v5
			if k6 then t[k6] = v6
			if k7 then t[k7] = v7
			if k8 then t[k8] = v8
			if k9 then t[k9] = v9
			if k10 then t[k10] = v10
			if k11 then t[k11] = v11
			if k12 then t[k12] = v12
			if k13 then t[k13] = v13
			if k14 then t[k14] = v14
			if k15 then t[k15] = v15
			if k16 then t[k16] = v16
			if k17 then t[k17] = v17
			if k18 then t[k18] = v18
			if k19 then t[k19] = v19
			if k20 then t[k20] = v20
			end end end end end end end end end end end end end end end end end end end end
		end
		return t
	end
	if lua51 then
		tmp = loadstring("local t = {}; return function(...) for k in pairs(t) do t[k] = nil end; for i = 1, select('#', ...), 2 do if select(i, ...) then t[select(i, ...)] = select(i+1, ...) else break; end; end; return t; end")()
	end
end
local tmp2
do
	local t = {}
	function tmp2(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20)
		for k in pairs(t) do
			t[k] = nil
		end
		if k1 then t[k1] = v1
		if k2 then t[k2] = v2
		if k3 then t[k3] = v3
		if k4 then t[k4] = v4
		if k5 then t[k5] = v5
		if k6 then t[k6] = v6
		if k7 then t[k7] = v7
		if k8 then t[k8] = v8
		if k9 then t[k9] = v9
		if k10 then t[k10] = v10
		if k11 then t[k11] = v11
		if k12 then t[k12] = v12
		if k13 then t[k13] = v13
		if k14 then t[k14] = v14
		if k15 then t[k15] = v15
		if k16 then t[k16] = v16
		if k17 then t[k17] = v17
		if k18 then t[k18] = v18
		if k19 then t[k19] = v19
		if k20 then t[k20] = v20
		end end end end end end end end end end end end end end end end end end end end
		return t
	end
	if lua51 then
		tmp2 = loadstring("local t = {}; return function(...) for k in pairs(t) do t[k] = nil end; for i = 1, select('#', ...), 2 do if select(i, ...) then t[select(i, ...)] = select(i+1, ...) else break; end; end; return t; end")()
	end
end
local levels
local buttons

local function GetScaledCursorPosition()
	local x, y = GetCursorPosition()
	local scale = UIParent:GetEffectiveScale()
	return x / scale, y / scale
end

local function StartCounting(self, levelNum)
	for i = levelNum, table.getn(levels) do
		if levels[i] then
			levels[i].count = 3
		end
	end
end

local function StopCounting(self, level)
	for i = level, 1, -1 do
		if levels[i] then
			levels[i].count = nil
		end
	end
end

local function OnUpdate(self, arg1)
	for _,level in ipairs(levels) do
		if level.count then
			level.count = level.count - arg1
			if level.count < 0 then
				level.count = nil
				self:Close(level.num)
			end
		end
	end
end

local function CheckDualMonitor(self, frame)
	local ratio = GetScreenWidth() / GetScreenHeight()
	if ratio >= 2.4 and frame:GetRight() > GetScreenWidth() / 2 and frame:GetLeft() < GetScreenWidth() / 2 then
		local offsetx
		if GetCursorPosition() / GetScreenHeight() * 768 < GetScreenWidth() / 2 then
			offsetx = GetScreenWidth() / 2 - frame:GetRight()
		else
			offsetx = GetScreenWidth() / 2 - frame:GetLeft()
		end
		local point, parent, relativePoint, x, y = frame:GetPoint(1)
		frame:SetPoint(point, parent, relativePoint, (x or 0) + offsetx, y or 0)
	end
end

local function CheckSize(self, level)
	if not level.buttons then
		return
	end
	local height = 20
	for _, button in ipairs(level.buttons) do
		height = height + button:GetHeight()
	end
	level:SetHeight(height)
	local width = 160
	for _, button in ipairs(level.buttons) do
		local extra = 1
		if button.hasArrow or button.hasColorSwatch then
			extra = extra + 16
		end
		if not button.notCheckable then
			extra = extra + 24
		end
		button.text:SetFont(STANDARD_TEXT_FONT, button.textHeight)
		if button.text:GetWidth() + extra > width then
			width = button.text:GetWidth() + extra
		end
	end
	level:SetWidth(width + 20)
	if level:GetLeft() and level:GetRight() and level:GetTop() and level:GetBottom() and (level:GetLeft() < 0 or level:GetRight() > GetScreenWidth() or level:GetTop() > GetScreenHeight() or level:GetBottom() < 0) then
		level:ClearAllPoints()
		if level.lastDirection == "RIGHT" then
			if level.lastVDirection == "DOWN" then
				level:SetPoint("TOPLEFT", level.parent or level:GetParent(), "TOPRIGHT", 5, 10)
			else
				level:SetPoint("BOTTOMLEFT", level.parent or level:GetParent(), "BOTTOMRIGHT", 5, -10)
			end
		else
			if level.lastVDirection == "DOWN" then
				level:SetPoint("TOPRIGHT", level.parent or level:GetParent(), "TOPLEFT", -5, 10)
			else
				level:SetPoint("BOTTOMRIGHT", level.parent or level:GetParent(), "BOTTOMLEFT", -5, -10)
			end
		end
	end
	local dirty = false
	if not level:GetRight() then
		self:Close()
		return
	end
	if level:GetRight() > GetScreenWidth() and level.lastDirection == "RIGHT" then
		level.lastDirection = "LEFT"
		dirty = true
	elseif level:GetLeft() < 0 and level.lastDirection == "LEFT" then
		level.lastDirection = "RIGHT"
		dirty = true
	end
	if level:GetTop() > GetScreenHeight() and level.lastVDirection == "UP" then
		level.lastVDirection = "DOWN"
		dirty = true
	elseif level:GetBottom() < 0 and level.lastVDirection == "DOWN" then
		level.lastVDirection = "UP"
		dirty = true
	end
	if dirty then
		level:ClearAllPoints()
		if level.lastDirection == "RIGHT" then
			if level.lastVDirection == "DOWN" then
				level:SetPoint("TOPLEFT", level.parent or level:GetParent(), "TOPRIGHT", 5, 10)
			else
				level:SetPoint("BOTTOMLEFT", level.parent or level:GetParent(), "BOTTOMRIGHT", 5, -10)
			end
		else
			if level.lastVDirection == "DOWN" then
				level:SetPoint("TOPRIGHT", level.parent or level:GetParent(), "TOPLEFT", -5, 10)
			else
				level:SetPoint("BOTTOMRIGHT", level.parent or level:GetParent(), "BOTTOMLEFT", -5, -10)
			end
		end
	end
	if level:GetTop() > GetScreenHeight() then
		local top = level:GetTop()
		local point, parent, relativePoint, x, y = level:GetPoint(1)
		level:ClearAllPoints()
		level:SetPoint(point, parent, relativePoint, x or 0, (y or 0) + GetScreenHeight() - top)
	elseif level:GetBottom() < 0 then
		local bottom = level:GetBottom()
		local point, parent, relativePoint, x, y = level:GetPoint(1)
		level:ClearAllPoints()
		level:SetPoint(point, parent, relativePoint, x or 0, (y or 0) - bottom)
	end
	CheckDualMonitor(self, level)
	if mod(level.num, 5) == 0 then
		local left, bottom = level:GetLeft(), level:GetBottom()
		level:ClearAllPoints()
		level:SetPoint("BOTTOMLEFT", UIParent, "BOTTOMLEFT", left, bottom)
	end
end

local Open
local OpenSlider
local OpenEditBox
local Refresh
local Clear
local function ReleaseButton(self, level, index)
	if not level.buttons then
		return
	end
	if not level.buttons[index] then
		return
	end
	local button = level.buttons[index]
	button:Hide()
	if button.highlight then
		button.highlight:Hide()
	end
--	button.arrow:SetVertexColor(1, 1, 1)
	button.arrow:SetHeight(16)
	button.arrow:SetWidth(16)
	table.remove(level.buttons, index)
	table.insert(buttons, button)
	for k in pairs(button) do
		if k ~= 0 and k ~= "text" and k ~= "check" and k ~= "arrow" and k ~= "colorSwatch" and k ~= "highlight" and k ~= "radioHighlight" then
			button[k] = nil
		end
	end
	return true
end

local function Scroll(self, level, down)
	if down then
		if level:GetBottom() < 0 then
			local point, parent, relativePoint, x, y = level:GetPoint(1)
			level:SetPoint(point, parent, relativePoint, x, y + 50)
			if level:GetBottom() > 0 then
				level:SetPoint(point, parent, relativePoint, x, y + 50 - level:GetBottom())
			end
		end
	else
		if level:GetTop() > GetScreenHeight() then
			local point, parent, relativePoint, x, y = level:GetPoint(1)
			level:SetPoint(point, parent, relativePoint, x, y - 50)
			if level:GetTop() < GetScreenHeight() then
				level:SetPoint(point, parent, relativePoint, x, y - 50 + GetScreenHeight() - level:GetTop())
			end
		end
	end
end

local sliderFrame
local editBoxFrame

local function AcquireButton(self, level)
	if not levels[level] then
		return
	end
	level = levels[level]
	if not level.buttons then
		level.buttons = {}
	end
	local button
	if table.getn(buttons) == 0 then
		button = CreateFrame("Button")
		button:SetFrameStrata("FULLSCREEN_DIALOG")
		button:SetHeight(16)
		local highlight = button:CreateTexture(nil, "BACKGROUND")
		highlight:SetTexture("Interface\\QuestFrame\\UI-QuestTitleHighlight")
		button.highlight = highlight
		highlight:SetBlendMode("ADD")
		highlight:SetAllPoints(button)
		highlight:Hide()
		local check = button:CreateTexture(nil, "ARTWORK")
		button.check = check
		check:SetTexture("Interface\\Buttons\\UI-CheckBox-Check")
		check:SetPoint("CENTER", button, "LEFT", 12, 0)
		check:SetWidth(24)
		check:SetHeight(24)
		local radioHighlight = button:CreateTexture(nil, "ARTWORK")
		button.radioHighlight = radioHighlight
		radioHighlight:SetTexture("Interface\\Buttons\\UI-RadioButton")
		radioHighlight:SetAllPoints(check)
		radioHighlight:SetBlendMode("ADD")
		radioHighlight:SetTexCoord(0.5, 0.75, 0, 1)
		radioHighlight:Hide()
		button:SetScript("OnEnter", function()
			local this = this
			if (sliderFrame and sliderFrame:IsShown() and sliderFrame.mouseDown and sliderFrame.level == this.level.num + 1) or (editBoxFrame and editBoxFrame:IsShown() and editBoxFrame.mouseDown and editBoxFrame.level == this.level.num + 1) then
				for i = 1, this.level.num do
					Refresh(self, levels[i])
				end
				return
			end
			self:Close(this.level.num + 1)
			if not this.disabled then
				if this.hasSlider then
					OpenSlider(self, this)
				elseif this.hasEditBox then
					OpenEditBox(self, this)
				elseif this.hasArrow then
					Open(self, this, nil, this.level.num + 1, this.value)
				end
			end
			if not this.level then -- button reclaimed
				return
			end
			StopCounting(self, this.level.num + 1)
			if not this.disabled then
				highlight:Show()
				if this.isRadio then
					button.radioHighlight:Show()
				end
			end
			if this.tooltipTitle or this.tooltipText then
				GameTooltip_SetDefaultAnchor(GameTooltip, this)
				local disabled = not this.isTitle and this.disabled
				if this.tooltipTitle then
					if disabled then
						GameTooltip:SetText(this.tooltipTitle, 0.5, 0.5, 0.5, 1)
					else
						GameTooltip:SetText(this.tooltipTitle, 1, 1, 1, 1)
					end
					if this.tooltipText then
						if disabled then
							GameTooltip:AddLine(this.tooltipText, (NORMAL_FONT_COLOR.r + 0.5) / 2, (NORMAL_FONT_COLOR.g + 0.5) / 2, (NORMAL_FONT_COLOR.b + 0.5) / 2, 1)
						else
							GameTooltip:AddLine(this.tooltipText, NORMAL_FONT_COLOR.r, NORMAL_FONT_COLOR.g, NORMAL_FONT_COLOR.b, 1)
						end
					end
				else
					if disabled then
						GameTooltip:SetText(this.tooltipText, 0.5, 0.5, 0.5, 1)
					else
						GameTooltip:SetText(this.tooltipText, 1, 1, 1, 1)
					end
				end
				GameTooltip:Show()
			end
			if this.tooltipFunc then
				GameTooltip:SetOwner(this, "ANCHOR_NONE")
				GameTooltip:SetPoint("TOPLEFT", this, "TOPRIGHT", 5, 0)
				this.tooltipFunc(this.tooltipArg1, this.tooltipArg2, this.tooltipArg3, this.tooltipArg4)
				GameTooltip:Show()
			end
		end)
		button:SetScript("OnLeave", function()
			highlight:Hide()
			button.radioHighlight:Hide()
			if this.level then
				StartCounting(self, this.level.num)
			end
			GameTooltip:Hide()
		end)
		button:SetScript("OnClick", function()
			if not this.disabled then
				if this.hasColorSwatch then
					local func = button.colorFunc
					local a1,a2,a3,a4 = button.colorArg1, button.colorArg2, button.colorArg3, button.colorArg4
					local hasOpacity = this.hasOpacity
					ColorPickerFrame.func = function()
						if func then
							local r,g,b = ColorPickerFrame:GetColorRGB()
							local a = hasOpacity and 1 - OpacitySliderFrame:GetValue() or nil
							if a1 == nil then
								func(r, g, b, a)
							elseif a2 == nil then
								func(a1, r, g, b, a)
							elseif a3 == nil then
								func(a1, a2, r, g, b, a)
							elseif a4 == nil then
								func(a1, a2, a3, r, g, b, a)
							else
								func(a1, a2, a3, a4, r, g, b, a)
							end
						end
					end
					ColorPickerFrame.hasOpacity = this.hasOpacity
					ColorPickerFrame.opacityFunc = ColorPickerFrame.func
					ColorPickerFrame.opacity = 1 - this.opacity
					ColorPickerFrame:SetColorRGB(this.r, this.g, this.b)
					local r, g, b, a = this.r, this.g, this.b, this.opacity
					ColorPickerFrame.cancelFunc = function()
						if a1 == nil then
							func(r, g, b, a)
						elseif a2 == nil then
							func(a1, r, g, b, a)
						elseif a3 == nil then
							func(a1, a2, r, g, b, a)
						else
							func(a1, a2, a3, r, g, b, a)
						end
					end
					self:Close(1)
					ShowUIPanel(ColorPickerFrame)
				elseif this.func then
					local level = button.level
					if type(this.func) == "string" then
						self:assert(type(this.arg1[this.func]) == "function", "Cannot call method " .. this.func)
						this.arg1[this.func](this.arg1, this.arg2, this.arg3, this.arg4)
					else
						this.func(this.arg1, this.arg2, this.arg3, this.arg4)
					end
					if this.closeWhenClicked then
						self:Close()
					elseif level:IsShown() then
						for i = 1, level.num do
							Refresh(self, levels[i])
						end
					end
				elseif this.closeWhenClicked then
					self:Close()
				end
			end
		end)
		local text = button:CreateFontString(nil, "ARTWORK")
		button.text = text
		text:SetFontObject(GameFontHighlightSmall)
		button.text:SetFont(STANDARD_TEXT_FONT, UIDROPDOWNMENU_DEFAULT_TEXT_HEIGHT)
		button:SetScript("OnMouseDown", function()
			if not this.disabled and (this.func or this.colorFunc or this.closeWhenClicked) then
				text:SetPoint("LEFT", button, "LEFT", this.notCheckable and 1 or 25, -1)
			end
		end)
		button:SetScript("OnMouseUp", function()
			if not this.disabled and (this.func or this.colorFunc or this.closeWhenClicked) then
				text:SetPoint("LEFT", button, "LEFT", this.notCheckable and 0 or 24, 0)
			end
		end)
		local arrow = button:CreateTexture(nil, "ARTWORK")
		button.arrow = arrow
		arrow:SetPoint("LEFT", button, "RIGHT", -16, 0)
		arrow:SetWidth(16)
		arrow:SetHeight(16)
		arrow:SetTexture("Interface\\ChatFrame\\ChatFrameExpandArrow")
		local colorSwatch = button:CreateTexture(nil, "OVERLAY")
		button.colorSwatch = colorSwatch
		colorSwatch:SetWidth(20)
		colorSwatch:SetHeight(20)
		colorSwatch:SetTexture("Interface\\ChatFrame\\ChatFrameColorSwatch")
		local texture = button:CreateTexture(nil, "OVERLAY")
		colorSwatch.texture = texture
		texture:SetTexture(1, 1, 1)
		texture:SetWidth(11.5)
		texture:SetHeight(11.5)
		texture:Show()
		texture:SetPoint("CENTER", colorSwatch, "CENTER")
		colorSwatch:SetPoint("RIGHT", button, "RIGHT", 0, 0)
	else
		button = buttons[table.getn(buttons)]
		table.remove(buttons, table.getn(buttons))
	end
	button:ClearAllPoints()
	button:SetParent(level)
	button:SetFrameStrata(level:GetFrameStrata())
	button:SetFrameLevel(level:GetFrameLevel() + 1)
	button:SetPoint("LEFT", level, "LEFT", 10, 0)
	button:SetPoint("RIGHT", level, "RIGHT", -10, 0)
	if table.getn(level.buttons) == 0 then
		button:SetPoint("TOP", level, "TOP", 0, -10)
	else
		button:SetPoint("TOP", level.buttons[table.getn(level.buttons)], "BOTTOM", 0, 0)
	end
	button.text:SetPoint("LEFT", button, "LEFT", 24, 0)
	button:Show()
	button.level = level
	table.insert(level.buttons, button)
	if not level.parented then
		level.parented = true
		level:ClearAllPoints()
		if level.num == 1 then
			if level.parent ~= UIParent then
				level:SetPoint("TOPRIGHT", level.parent, "TOPLEFT")
			else
				level:SetPoint("CENTER", level.parent, "CENTER")
			end
		else
			if level.lastDirection == "RIGHT" then
				if level.lastVDirection == "DOWN" then
					level:SetPoint("TOPLEFT", level.parent, "TOPRIGHT", 5, 10)
				else
					level:SetPoint("BOTTOMLEFT", level.parent, "BOTTOMRIGHT", 5, -10)
				end
			else
				if level.lastVDirection == "DOWN" then
					level:SetPoint("TOPRIGHT", level.parent, "TOPLEFT", -5, 10)
				else
					level:SetPoint("BOTTOMRIGHT", level.parent, "BOTTOMLEFT", -5, -10)
				end
			end
		end
		level:SetFrameStrata("FULLSCREEN_DIALOG")
	end
	return button
end

local function AcquireLevel(self, level)
	if not levels[level] then
		for i = table.getn(levels) + 1, level, -1 do
			local i = i
			local frame = CreateFrame("Button")
			if i == 1 then
				local old_CloseWindows = CloseWindows
				function CloseWindows(ignoreCenter)
					local found = old_CloseWindows(ignoreCenter)
					if levels[1]:IsShown() then
						self:Close()
						return 1
					end
					return found
				end
			end
			levels[i] = frame
			frame.num = i
			frame:SetParent(UIParent)
			frame:SetFrameStrata("FULLSCREEN_DIALOG")
			frame:Hide()
			frame:SetWidth(180)
			frame:SetHeight(10)
			frame:SetFrameLevel(i * 3)
			frame:SetScript("OnHide", function()
				self:Close(level + 1)
			end)
			frame:SetFrameStrata("FULLSCREEN_DIALOG")
			if frame.SetTopLevel then
				frame:SetTopLevel(true)
			end
			frame:EnableMouse(true)
			frame:EnableMouseWheel(true)
			local backdrop = CreateFrame("Frame", nil, frame)
			backdrop:SetAllPoints(frame)
			backdrop:SetBackdrop(tmp(
				'bgFile', "Interface\\Tooltips\\UI-Tooltip-Background",
				'edgeFile', "Interface\\Tooltips\\UI-Tooltip-Border",
				'tile', true,
				'insets', tmp2(
					'left', 5,
					'right', 5,
					'top', 5,
					'bottom', 5
				),
				'tileSize', 16,
				'edgeSize', 16
			))
			backdrop:SetBackdropBorderColor(TOOLTIP_DEFAULT_COLOR.r, TOOLTIP_DEFAULT_COLOR.g, TOOLTIP_DEFAULT_COLOR.b)
			backdrop:SetBackdropColor(TOOLTIP_DEFAULT_BACKGROUND_COLOR.r, TOOLTIP_DEFAULT_BACKGROUND_COLOR.g, TOOLTIP_DEFAULT_BACKGROUND_COLOR.b)
			frame:SetScript("OnClick", function()
				self:Close(i)
			end)
			frame:SetScript("OnEnter", function()
				StopCounting(self, i)
			end)
			frame:SetScript("OnLeave", function()
				StartCounting(self, i)
			end)
			frame:SetScript("OnMouseWheel", function()
				Scroll(self, frame, arg1 < 0)
			end)
			if i == 1 then
				frame:SetScript("OnUpdate", function()
					OnUpdate(self, arg1)
				end)
				levels[1].lastDirection = "RIGHT"
				levels[1].lastVDirection = "DOWN"
			else
				levels[i].lastDirection = levels[i - 1].lastDirection
				levels[i].lastVDirection = levels[i - 1].lastVDirection
			end
		end
	end
	return levels[level]
end

local function checkValidate(validateFunc, func, arg1, arg2, arg3)
	local text
	if arg3 ~= nil then
		text = arg3
	elseif arg2 ~= nil then
		text = arg2
	else
		text = arg1
	end
	if not validateFunc(text) then
		DEFAULT_CHAT_FRAME:AddMessage("|cffffff7fValidation error: [|r" .. tostring(text) .. "|cffffff7f]|r")
	else
		func(arg1, arg2, arg3)
	end
end

local function validateOptions(options, position, baseOptions, fromPass)
	if not baseOptions then
		baseOptions = options
	end
	if type(options) ~= "table" then
		return "Options must be a table.", position
	end
	local kind = options.type
	if type(kind) ~= "string" then
		return '"type" must be a string.', position
	elseif kind ~= "group" and kind ~= "range" and kind ~= "text" and kind ~= "execute" and kind ~= "toggle" and kind ~= "color" and kind ~= "header" then
		return '"type" must either be "range", "text", "group", "toggle", "execute", "color", or "header".', position
	end
	if options.aliases then
		if type(options.aliases) ~= "table" and type(options.aliases) ~= "string" then
			return '"alias" must be a table or string', position
		end
	end
	if not fromPass then
		if kind == "execute" then
			if type(options.func) ~= "string" and type(options.func) ~= "function" then
				return '"func" must be a string or function', position
			end
		elseif kind == "range" or kind == "text" or kind == "toggle" then
			if type(options.set) ~= "string" and type(options.set) ~= "function" then
				return '"set" must be a string or function', position
			end
			if kind == "text" and options.get == false then
			elseif type(options.get) ~= "string" and type(options.get) ~= "function" then
				return '"get" must be a string or function', position
			end
		elseif kind == "group" and options.pass then
			if options.pass ~= true then
				return '"pass" must be either nil, true, or false', position
			end
			if not options.func then
				if type(options.set) ~= "string" and type(options.set) ~= "function" then
					return '"set" must be a string or function', position
				end
				if type(options.get) ~= "string" and type(options.get) ~= "function" then
					return '"get" must be a string or function', position
				end
			elseif type(options.func) ~= "string" and type(options.func) ~= "function" then
				return '"func" must be a string or function', position
			end
		end
	else
		if kind == "group" then
			return 'cannot have "type" = "group" as a subgroup of a passing group', position
		end
	end
	if options ~= baseOptions then
		if kind == "header" then
		elseif type(options.desc) ~= "string" then
			return '"desc" must be a string', position
		elseif string.len(options.desc) == 0 then
			return '"desc" cannot be a 0-length string', position
		end
	end
	if options ~= baseOptions or kind == "range" or kind == "text" or kind == "toggle" or kind == "color" then
		if options.type == "header" and not options.cmdName and not options.name then
		elseif options.cmdName then
			if type(options.cmdName) ~= "string" then
				return '"cmdName" must be a string or nil', position
			elseif string.len(options.cmdName) == 0 then
				return '"cmdName" cannot be a 0-length string', position
			end
			if type(options.guiName) ~= "string" then
				if not options.guiNameIsMap then
					return '"guiName" must be a string or nil', position
				end
			elseif string.len(options.guiName) == 0 then
				return '"guiName" cannot be a 0-length string', position
			end
		else
			if type(options.name) ~= "string" then
				return '"name" must be a string', position
			elseif string.len(options.name) == 0 then
				return '"name" cannot be a 0-length string', position
			end
		end
	end
	if options.guiNameIsMap then
		if type(options.guiNameIsMap) ~= "boolean" then
			return '"guiNameIsMap" must be a boolean or nil', position
		elseif options.type ~= "toggle" then
			return 'if "guiNameIsMap" is true, then "type" must be set to \'toggle\'', position
		elseif type(options.map) ~= "table" then
			return '"map" must be a table', position
		end
	end
	if options.message and type(options.message) ~= "string" then
		return '"message" must be a string or nil', position
	end
	if options.error and type(options.error) ~= "string" then
		return '"error" must be a string or nil', position
	end
	if options.current and type(options.current) ~= "string" then
		return '"current" must be a string or nil', position
	end
	if options.order then
		if type(options.order) ~= "number" or (-1 < options.order and options.order < 0.999) then
			return '"order" must be a non-zero number or nil', position
		end
	end
	if options.disabled then
		if type(options.disabled) ~= "function" and type(options.disabled) ~= "string" and options.disabled ~= true then
			return '"disabled" must be a function, string, or boolean', position
		end
	end
	if options.cmdHidden then
		if type(options.cmdHidden) ~= "function" and type(options.cmdHidden) ~= "string" and options.cmdHidden ~= true then
			return '"cmdHidden" must be a function, string, or boolean', position
		end
	end
	if options.guiHidden then
		if type(options.guiHidden) ~= "function" and type(options.guiHidden) ~= "string" and options.guiHidden ~= true then
			return '"guiHidden" must be a function, string, or boolean', position
		end
	end
	if options.hidden then
		if type(options.hidden) ~= "function" and type(options.hidden) ~= "string" and options.hidden ~= true then
			return '"hidden" must be a function, string, or boolean', position
		end
	end
	if kind == "text" then
		if type(options.validate) == "table" then
			local t = options.validate
			local iTable = nil
			for k,v in pairs(t) do
				if type(k) == "number" then
					if iTable == nil then
						iTable = true
					elseif not iTable then
						return '"validate" must either have all keys be indexed numbers or strings', position
					elseif k < 1 or k > table.getn(t) then
						return '"validate" numeric keys must be indexed properly. >= 1 and <= table.getn', position
					end
				else
					if iTable == nil then
						iTable = false
					elseif iTable then
						return '"validate" must either have all keys be indexed numbers or strings', position
					end
				end
				if type(v) ~= "string" then
					return '"validate" values must all be strings', position
				end
			end
		else
			if type(options.usage) ~= "string" then
				return '"usage" must be a string', position
			elseif options.validate and type(options.validate) ~= "string" and type(options.validate) ~= "function" then
				return '"validate" must be a string, function, or table', position
			end
		end
	elseif kind == "range" then
		if options.min or options.max then
			if type(options.min) ~= "number" then
				return '"min" must be a number', position
			elseif type(options.max) ~= "number" then
				return '"max" must be a number', position
			elseif options.min >= options.max then
				return '"min" must be less than "max"', position
			end
		end
		if options.step then
			if type(options.step) ~= "number" then
				return '"step" must be a number', position
			elseif options.step < 0 then
				return '"step" must be nonnegative', position
			end
		end
		if options.isPercent and options.isPercent ~= true then
			return '"isPercent" must either be nil, true, or false', position
		end
	elseif kind == "toggle" then
		if options.map then
			if type(options.map) ~= "table" then
				return '"map" must be a table', position
			elseif type(options.map[true]) ~= "string" then
				return '"map[true]" must be a string', position
			elseif type(options.map[false]) ~= "string" then
				return '"map[false]" must be a string', position
			end
		end
	elseif kind == "color" then
		if options.hasAlpha and options.hasAlpha ~= true then
			return '"hasAlpha" must be nil, true, or false', position
		end
	elseif kind == "group" then
		if options.pass and options.pass ~= true then
			return '"pass" must be nil, true, or false', position
		end
		if type(options.args) ~= "table" then
			return '"args" must be a table', position
		end
		for k,v in pairs(options.args) do
			if type(k) ~= "string" then
				return '"args" keys must be strings', position
			elseif string.find(k, "%s") then
				return string.format('"args" keys must not include spaces. %q is not appropriate.', k), position
			elseif string.len(k) == 0 then
				return '"args" keys must not be 0-length strings.', position
			end
			if type(v) ~= "table" then
				return '"args" values must be tables', position and position .. "." .. k or k
			end
			local newposition
			if position then
				newposition = position .. ".args." .. k
			else
				newposition = "args." .. k
			end
			local err, pos = validateOptions(v, newposition, baseOptions, options.pass)
			if err then
				return err, pos
			end
		end
	end
	if options.icon and type(options.icon) ~= "string" then
		return'"icon" must be a string', position
	end
	if options.iconWidth or options.iconHeight then
		if (options.iconWidth and type(options.iconWidth) ~= "number") or (options.iconHeight and type(options.iconHeight) ~= "number") then
			return '"iconHeight" and "iconWidth" must be numbers', position
		end
	end
end

local validatedOptions

local values
local mysort_args
local mysort
local othersort
local othersort_validate

local baseFunc, currentLevel

function Dewdrop:FeedAceOptionsTable(options, difference)
	self:argCheck(options, 2, "table")
	self:argCheck(difference, 3, "nil", "number")
	self:assert(currentLevel, "Cannot call `FeedAceOptionsTable' outside of a Dewdrop declaration")
	if not difference then
		difference = 0
	end
	if not validatedOptions then
		validatedOptions = {}
	end
	if not validatedOptions[options] then
		local err, position = validateOptions(options)
		
		if err then
			if position then
				Dewdrop:error(position .. ": " .. err)
			else
				Dewdrop:error(err)
			end
		end
		
		validatedOptions[options] = true
	end
	local level = levels[currentLevel]
	self:assert(level, "Improper level given")
	if not values then
		values = {}
	else
		for k,v in pairs(values) do
			values[k] = nil
		end
		table_setn(values, 0)
	end
	
	local current = level
	while current do
		if current.num == difference + 1 then
			break
		end
		table.insert(values, current.value)
		current = levels[current.num - 1]
	end
	
	local realOptions = options
	local handler = options.handler
	local passTable
	local passValue
	while table.getn(values) > 0 do
		passTable = options.pass and current or nil
		local value = table.remove(values)
		options = options.args and options.args[value]
		if not options then
			return
		end
		handler = options.handler or handler
		passValue = passTable and value or nil
	end
	
	if options.type == "group" then
		for k in pairs(options.args) do
			table.insert(values, k)
		end
		if not mysort then
			mysort = function(a, b)
				local alpha, bravo = mysort_args[a], mysort_args[b]
				local alpha_order = alpha.order or 100
				local bravo_order = bravo.order or 100
				local alpha_name = alpha.guiName or alpha.name
				local bravo_name = bravo.guiName or bravo.name
				if alpha_order == bravo_order then
					if not alpha_name then
						return true
					elseif not bravo_name then
						return false
					else
						return string.upper(alpha_name) < string.upper(bravo_name)
					end
				else
					if alpha_order < 0 then
						if bravo_order > 0 then
							return false
						end
					else
						if bravo_order < 0 then
							return true
						end
					end
					return alpha_order < bravo_order
				end
			end
		end
		mysort_args = options.args
		table.sort(values, mysort)
		mysort_args = nil
		local hasBoth = table.getn(values) >= 1 and (options.args[values[1]].order or 100) > 0 and (options.args[values[table.getn(values)]].order or 100) < 0
		local last_order = 1
		for _,k in ipairs(values) do
			local v = options.args[k]
			local handler = v.handler or handler
			if hasBoth and last_order > 0 and (v.order or 100) < 0 then
				hasBoth = false
				self:AddLine()
			end
			local hidden, disabled = v.guiHidden or v.hidden, v.disabled
			if type(hidden) == "function" then
				hidden = hidden()
			elseif type(hidden) == "string" then
				hidden = handler[hidden](handler)
			end
			if not hidden then
				if type(disabled) == "function" then
					disabled = disabled()
				elseif type(disabled) == "string" then
					disabled = handler[disabled](handler)
				end
				local name = (v.guiIconOnly and v.icon) and "" or (v.guiName or v.name)
				local desc = v.desc
				local iconHeight = v.iconHeight or 16
				local iconWidth = v.iconWidth or 16
				local tooltipTitle, tooltipText
				tooltipTitle = name
				if name ~= desc then
					tooltipText = desc
				end
				if v.type == "toggle" then
					local checked
					if type(v.get) == "function" then
						checked = v.get(passValue)
					else
						if not handler[v.get] then
							Dewdrop:error("Handler %q not available", v.get)
						end
						checked = handler[v.get](handler, passValue)
					end
					local func, arg1, arg2, arg3
					if type(v.set) == "function" then
						func = v.set
						if passValue ~= nil then
							arg1 = passValue
							arg2 = not checked
						else
							arg1 = not checked
						end
					else
						if not handler[v.set] then
							Dewdrop:error("Handler %q not available", v.set)
						end
						func = handler[v.set]
						arg1 = handler
						if passValue ~= nil then
							arg2 = passValue
							arg3 = not checked
						else
							arg2 = not checked
						end
					end
					if v.guiNameIsMap then
						checked = checked and true or false
						name = string.gsub(tostring(v.map and v.map[checked]), "|c%x%x%x%x%x%x%x%x(.-)|r", "%1")
						checked = nil
					end
					self:AddLine(
						'text', name,
						'checked', checked,
						'isRadio', v.isRadio,
						'func', func,
						'arg1', arg1,
						'arg2', arg2,
						'arg3', arg3,
						'disabled', disabled,
						'tooltipTitle', tooltipTitle,
						'tooltipText', tooltipText
					)
				elseif v.type == "execute" then
					local func, arg1, arg2
					if type(v.func) == "function" then
						func = v.func
						arg1 = passValue
					else
						if not handler[v.func] then
							Dewdrop:error("Handler %q not available", v.func)
						end
						func = handler[v.func]
						arg1 = handler
						arg2 = passValue
					end
					self:AddLine(
						'text', name,
						'checked', checked,
						'func', func,
						'arg1', arg1,
						'arg2', arg2,
						'disabled', disabled,
						'tooltipTitle', tooltipTitle,
						'tooltipText', tooltipText,
						'icon', v.icon,
						'iconHeight', iconHeight,
						'iconWidth', iconWidth
					)
				elseif v.type == "range" then
					local sliderValue
					if type(v.get) == "function" then
						sliderValue = v.get(passValue)
					else
						if not handler[v.get] then
							Dewdrop:error("Handler %q not available", v.get)
						end
						sliderValue = handler[v.get](handler, passValue)
					end
					local sliderFunc, sliderArg1, sliderArg2
					if type(v.set) == "function" then
						sliderFunc = v.set
						sliderArg1 = passValue
					else
						if not handler[v.set] then
							Dewdrop:error("Handler %q not available", v.set)
						end
						sliderFunc = handler[v.set]
						sliderArg1 = handler
						sliderArg2 = passValue
					end
					self:AddLine(
						'text', name,
						'hasArrow', true,
						'hasSlider', true,
						'sliderMin', v.min or 0,
						'sliderMax', v.max or 1,
						'sliderStep', v.step or 0,
						'sliderIsPercent', v.isPercent or false,
						'sliderValue', sliderValue,
						'sliderFunc', sliderFunc,
						'sliderArg1', sliderArg1,
						'sliderArg2', sliderArg2,
						'disabled', disabled,
						'tooltipTitle', tooltipTitle,
						'tooltipText', tooltipText,
						'icon', v.icon,
						'iconHeight', iconHeight,
						'iconWidth', iconWidth
					)
				elseif v.type == "color" then
					local r,g,b,a
					if type(v.get) == "function" then
						r,g,b,a = v.get(passValue)
					else
						if not handler[v.get] then
							Dewdrop:error("Handler %q not available", v.get)
						end
						r,g,b,a = handler[v.get](handler, passValue)
					end
					local colorFunc, colorArg1, colorArg2
					if type(v.set) == "function" then
						colorFunc = v.set
						colorArg1 = passValue
					else
						if not handler[v.set] then
							Dewdrop:error("Handler %q not available", v.set)
						end
						colorFunc = handler[v.set]
						colorArg1 = handler
						colorArg2 = passValue
					end
					self:AddLine(
						'text', name,
						'hasArrow', true,
						'hasColorSwatch', true,
						'r', r,
						'g', g,
						'b', b,
						'opacity', v.hasAlpha and a or nil,
						'hasOpacity', v.hasAlpha,
						'colorFunc', colorFunc,
						'colorArg1', colorArg1,
						'colorArg2', colorArg2,
						'disabled', disabled,
						'tooltipTitle', tooltipTitle,
						'tooltipText', tooltipText
					)
				elseif v.type == "text" then
					if type(v.validate) == "table" then
						self:AddLine(
							'text', name,
							'hasArrow', true,
							'value', k,
							'disabled', disabled,
							'tooltipTitle', tooltipTitle,
							'tooltipText', tooltipText,
							'icon', v.icon,
							'iconHeight', iconHeight,
							'iconWidth', iconWidth
						)
					else
						local editBoxText
						if type(v.get) == "function" then
							editBoxText = v.get(passValue)
						elseif v.get == false then
							editBoxText = nil
						else
							if not handler[v.get] then
								Dewdrop:error("Handler %q not available", v.get)
							end
							editBoxText = handler[v.get](handler, passValue)
						end
						local editBoxFunc, editBoxArg1, editBoxArg2
						if type(v.set) == "function" then
							editBoxFunc = v.set
							editBoxArg1 = passValue
						else
							if not handler[v.set] then
								Dewdrop:error("Handler %q not available", v.set)
							end
							editBoxFunc = handler[v.set]
							editBoxArg1 = handler
							editBoxArg2 = passValue
						end
						
						local editBoxValidateFunc, editBoxValidateArg1
						
						if v.validate then
							if type(v.validate) == "function" then
								editBoxValidateFunc = v.validate
							else
								if not handler[v.validate] then
									Dewdrop:error("Handler %q not available", v.validate)
								end
								editBoxValidateFunc = handler[v.validate]
								editBoxValidateArg1 = handler
							end
						end
						
						self:AddLine(
							'text', name,
							'hasArrow', true,
							'icon', v.icon,
							'iconHeight', iconHeight,
							'iconWidth', iconWidth,
							'hasEditBox', true,
							'editBoxText', editBoxText,
							'editBoxFunc', editBoxFunc,
							'editBoxArg1', editBoxArg1,
							'editBoxArg2', editBoxArg2,
							'editBoxValidateFunc', editBoxValidateFunc,
							'editBoxValidateArg1', editBoxValidateArg1,
							'disabled', disabled,
							'tooltipTitle', tooltipTitle,
							'tooltipText', tooltipText
						)
					end
				elseif v.type == "group" then
					self:AddLine(
						'text', name,
						'hasArrow', true,
						'value', k,
						'disabled', disabled,
						'tooltipTitle', tooltipTitle,
						'tooltipText', tooltipText,
						'icon', v.icon,
						'iconHeight', iconHeight,
						'iconWidth', iconWidth
					)
				elseif v.type == "header" then
					if name == "" or not name then
						self:AddLine(
							'isTitle', true,
							'icon', v.icon,
							'iconHeight', iconHeight,
							'iconWidth', iconWidth
						)
					else
						self:AddLine(
							'text', name,
							'isTitle', true,
							'icon', v.icon,
							'iconHeight', iconHeight,
							'iconWidth', iconWidth
						)
					end
				end
			end
			last_order = v.order or 100
		end
	elseif options.type == "text" and type(options.validate) == "table" then
		local current
		if type(options.get) == "function" then
			current = options.get(passValue)
		else
			if not handler[options.get] then
				Dewdrop:error("Handler %q not available", options.get)
			end
			current = handler[options.get](handler, passValue)
		end
		local indexed = true
		for k,v in pairs(options.validate) do
			if type(k) ~= "number" then
				indexed = false
			end
			table.insert(values, k)
		end
		if not indexed then
			if not othersort then
				othersort = function(alpha, bravo)
					return othersort_validate[alpha] < othersort_validate[bravo]
				end
			end
			othersort_validate = options.validate
			table.sort(values, othersort)
			othersort_validate = nil
		end
		for _,k in ipairs(values) do
			local v = options.validate[k]
			if type(k) == "number" then
				k = v
			end
			local func, arg1, arg2
			if type(options.set) == "function" then
				func = options.set
				if passValue ~= nil then
					arg1 = passValue
					arg2 = k
				else
					arg1 = k
				end
			else
				if not handler[options.set] then
					Dewdrop:error("Handler %q not available", options.set)
				end
				func = handler[options.set]
				arg1 = handler
				if passValue ~= nil then
					arg2 = passValue
					arg3 = k
				else
					arg2 = k
				end
			end
			local checked = (k == current or (type(k) == "string" and type(current) == "string" and string.lower(k) == string.lower(current)))
			self:AddLine(
				'text', v,
				'func', not checked and func or nil,
				'arg1', not checked and arg1 or nil,
				'arg2', not checked and arg2 or nil,
				'arg3', not checked and arg3 or nil,
				'isRadio', true,
				'checked',  checked,
				'tooltipTitle', options.guiName or options.name,
				'tooltipText', v
			)
		end
		for k in pairs(values) do
			values[k] = nil
		end
		table_setn(values, 0)
	else
		return false
	end
	return true
end

function Refresh(self, level)
	if type(level) == "number" then
		level = levels[level]
	end
	if not level then
		return
	end
	if baseFunc then
		Clear(self, level)
		currentLevel = level.num
		if type(baseFunc) == "table" then
			if currentLevel == 1 then
				local handler = baseFunc.handler
				if handler then
					local name = tostring(handler)
					if not string.find(name, '^table:') then
						name = string.gsub(name, "|c%x%x%x%x%x%x%x%x(.-)|r", "%1")
						self:AddLine(
							'text', name,
							'isTitle', true
						)
					end
				end
--			elseif level.parentText then
--				self:AddLine(
--					'text', level.parentText,
--					'tooltipTitle', level.parentTooltipTitle,
--					'tooltipText', level.parentTooltipText,
--					'tooltipFunc', level.parentTooltipFunc,
--					'isTitle', true
--				)
			end
			self:FeedAceOptionsTable(baseFunc)
			if currentLevel == 1 then
				self:AddLine(
					'text', CLOSE,
					'closeWhenClicked', true
				)
			end
		else
--			if level.parentText then
--				self:AddLine(
--					'text', level.parentText,
--					'tooltipTitle', level.parentTooltipTitle,
--					'tooltipText', level.parentTooltipText,
--					'tooltipFunc', level.parentTooltipFunc,
--					'isTitle', true
--				)
--			end
			baseFunc(currentLevel, level.value, levels[level.num - 1] and levels[level.num - 1].value, levels[level.num - 2] and levels[level.num - 2].value, levels[level.num - 3] and levels[level.num - 3].value, levels[level.num - 4] and levels[level.num - 4].value)
		end
		currentLevel = nil
		CheckSize(self, level)
	end
end

function Dewdrop:Refresh(level)
	self:argCheck(level, 2, "number")
	Refresh(self, levels[level])
end

function OpenSlider(self, parent)
	if not sliderFrame then
		sliderFrame = CreateFrame("Frame", nil, UIParent)
		sliderFrame:SetWidth(80)
		sliderFrame:SetHeight(170)
		sliderFrame:SetBackdrop(tmp(
			'bgFile', "Interface\\Tooltips\\UI-Tooltip-Background",
			'edgeFile', "Interface\\Tooltips\\UI-Tooltip-Border",
			'tile', true,
			'insets', tmp2(
				'left', 5,
				'right', 5,
				'top', 5,
				'bottom', 5
			),
			'tileSize', 16,
			'edgeSize', 16
		))
		sliderFrame:SetFrameStrata("FULLSCREEN_DIALOG")
		if sliderFrame.SetTopLevel then
			sliderFrame:SetTopLevel(true)
		end
		sliderFrame:SetBackdropBorderColor(TOOLTIP_DEFAULT_COLOR.r, TOOLTIP_DEFAULT_COLOR.g, TOOLTIP_DEFAULT_COLOR.b)
		sliderFrame:SetBackdropColor(TOOLTIP_DEFAULT_BACKGROUND_COLOR.r, TOOLTIP_DEFAULT_BACKGROUND_COLOR.g, TOOLTIP_DEFAULT_BACKGROUND_COLOR.b)
		sliderFrame:EnableMouse(true)
		sliderFrame:Hide()
		sliderFrame:SetPoint("CENTER", UIParent, "CENTER")
		local slider = CreateFrame("Slider", nil, sliderFrame)
		sliderFrame.slider = slider
		slider:SetOrientation("VERTICAL")
		slider:SetMinMaxValues(0, 1)
		slider:SetValueStep(0.01)
		slider:SetValue(0.5)
		slider:SetWidth(16)
		slider:SetHeight(128)
		slider:SetPoint("LEFT", sliderFrame, "LEFT", 15, 0)
		slider:SetBackdrop(tmp(
			'bgFile', "Interface\\Buttons\\UI-SliderBar-Background",
			'edgeFile', "Interface\\Buttons\\UI-SliderBar-Border",
			'tile', true,
			'edgeSize', 8,
			'tileSize', 8,
			'insets', tmp2(
				'left', 3,
				'right', 3,
				'top', 3,
				'bottom', 3
			)
		))
		local texture = slider:CreateTexture()
		slider:SetThumbTexture("Interface\\Buttons\\UI-SliderBar-Button-Vertical")
		local text = slider:CreateFontString(nil, "ARTWORK")
		sliderFrame.topText = text
		text:SetFontObject(GameFontGreenSmall)
		text:SetText("100%")
		text:SetPoint("BOTTOM", slider, "TOP")
		local text = slider:CreateFontString(nil, "ARTWORK")
		sliderFrame.bottomText = text
		text:SetFontObject(GameFontGreenSmall)
		text:SetText("0%")
		text:SetPoint("TOP", slider, "BOTTOM")
		local text = slider:CreateFontString(nil, "ARTWORK")
		sliderFrame.currentText = text
		text:SetFontObject(GameFontHighlightSmall)
		text:SetText("50%")
		text:SetPoint("LEFT", slider, "RIGHT")
		text:SetPoint("RIGHT", sliderFrame, "RIGHT", -6, 0)
		text:SetJustifyH("CENTER")
		local changed = false
		local inside = false
		slider:SetScript("OnValueChanged", function()
			if sliderFrame.changing then
				return
			end
			changed = true
			local done = false
			if sliderFrame.parent then
				if sliderFrame.parent.sliderFunc then
					local min = sliderFrame.parent.sliderMin or 0
					local max = sliderFrame.parent.sliderMax or 1
					local step = sliderFrame.parent.sliderStep or (max - min) / 100
					local a1,a2,a3,a4 = sliderFrame.parent.sliderArg1, sliderFrame.parent.sliderArg2, sliderFrame.parent.sliderArg3, sliderFrame.parent.sliderArg4
					local value = (1 - slider:GetValue()) * (max - min) + min
					if step > 0 then
						value = math.floor((value - min) / step + 0.5) * step + min
						if value > max then
							value = max
						elseif value < min then
							value = min
						end
					end
					local text
					if a1 == nil then
						text = sliderFrame.parent.sliderFunc(value)
					elseif a2 == nil then
						text = sliderFrame.parent.sliderFunc(a1, value)
					elseif a3 == nil then
						text = sliderFrame.parent.sliderFunc(a1, a2, value)
					elseif a4 == nil then
						text = sliderFrame.parent.sliderFunc(a1, a2, a3, value)
					else
						text = sliderFrame.parent.sliderFunc(a1, a2, a3, a4, value)
					end
					if text then
						sliderFrame.currentText:SetText(text)
						done = true
					end
				end
			end
			if not done then
				local min = sliderFrame.parent.sliderMin or 0
				local max = sliderFrame.parent.sliderMax or 1
				local step = sliderFrame.parent.sliderStep or (max - min) / 100
				local value = (1 - slider:GetValue()) * (max - min) + min
				if step > 0 then
					value = math.floor((value - min) / step + 0.5) * step + min
					if value > max then
						value = max
					elseif value < min then
						value = min
					end
				end
				if sliderFrame.parent.sliderIsPercent then
					sliderFrame.currentText:SetText(string.format("%.0f%%", value * 100))
				else
					if step < 0.1 then
						sliderFrame.currentText:SetText(string.format("%.2f", value))
					elseif step < 1 then
						sliderFrame.currentText:SetText(string.format("%.1f", value))
					else
						sliderFrame.currentText:SetText(string.format("%.0f", value))
					end
				end
			end
		end)
		sliderFrame:SetScript("OnEnter", function()
			StopCounting(self, sliderFrame.level)
		end)
		sliderFrame:SetScript("OnLeave", function()
			StartCounting(self, sliderFrame.level)
		end)
		slider:SetScript("OnMouseDown", function()
			sliderFrame.mouseDown = true
		end)
		slider:SetScript("OnMouseUp", function()
			sliderFrame.mouseDown = false
			if changed and not inside then
				local parent = sliderFrame.parent
				for i = 1, sliderFrame.level - 1 do
					Refresh(self, levels[i])
				end
				OpenSlider(self, parent)
			end
		end)
		slider:SetScript("OnEnter", function()
			inside = true
			StopCounting(self, sliderFrame.level)
		end)
		slider:SetScript("OnLeave", function()
			inside = false
			StartCounting(self, sliderFrame.level)
			if changed and not sliderFrame.mouseDown then
				local parent = sliderFrame.parent
				for i = 1, sliderFrame.level - 1 do
					Refresh(self, levels[i])
				end
				OpenSlider(self, parent)
			end
		end)
	end
	sliderFrame.parent = parent
	sliderFrame.level = parent.level.num + 1
	sliderFrame.parentValue = parent.level.value
	sliderFrame:SetFrameLevel(parent.level:GetFrameLevel() + 3)
	sliderFrame.slider:SetFrameLevel(sliderFrame:GetFrameLevel() + 1)
	sliderFrame.changing = true
	if not parent.sliderMin or not parent.sliderMax then
		return
	end
	
	if parent.arrow then
--		parent.arrow:SetVertexColor(0.2, 0.6, 0)
		parent.arrow:SetHeight(24)
		parent.arrow:SetWidth(24)
	end
	
	if not parent.sliderValue then
		parent.sliderValue = (parent.sliderMin + parent.sliderMax) / 2
	end
	sliderFrame.slider:SetValue(1 - (parent.sliderValue - parent.sliderMin) / (parent.sliderMax - parent.sliderMin))
	sliderFrame.changing = false
	sliderFrame.bottomText:SetText(parent.sliderMinText or "0")
	sliderFrame.topText:SetText(parent.sliderMaxText or "1")
	local text
	if parent.sliderFunc then
		local a1,a2,a3,a4 = parent.sliderArg1, parent.sliderArg2, parent.sliderArg3, parent.sliderArg4
		if a1 == nil then
			text = parent.sliderFunc(parent.sliderValue)
		elseif a2 == nil then
			text = parent.sliderFunc(a1, parent.sliderValue)
		elseif a3 == nil then
			text = parent.sliderFunc(a1, a2, parent.sliderValue)
		elseif a4 == nil then
			text = parent.sliderFunc(a1, a2, a3, parent.sliderValue)
		else
			text = parent.sliderFunc(a1, a2, a3, a4, parent.sliderValue)
		end
	end
	if text then
		sliderFrame.currentText:SetText(text)
	elseif parent.sliderIsPercent then
		sliderFrame.currentText:SetText(string.format("%.0f%%", parent.sliderValue * 100))
	else
		sliderFrame.currentText:SetText(parent.sliderValue)
	end
	
	local level = parent.level
	sliderFrame:Show()
	sliderFrame:ClearAllPoints()
	if level.lastDirection == "RIGHT" then
		if level.lastVDirection == "DOWN" then
			sliderFrame:SetPoint("TOPLEFT", parent, "TOPRIGHT", 5, 10)
		else
			sliderFrame:SetPoint("BOTTOMLEFT", parent, "BOTTOMRIGHT", 5, -10)
		end
	else
		if level.lastVDirection == "DOWN" then
			sliderFrame:SetPoint("TOPRIGHT", parent, "TOPLEFT", -5, 10)
		else
			sliderFrame:SetPoint("BOTTOMRIGHT", parent, "BOTTOMLEFT", -5, -10)
		end
	end
	local dirty
	if level.lastDirection == "RIGHT" then
		if sliderFrame:GetRight() > GetScreenWidth() then
			level.lastDirection = "LEFT"
			dirty = true
		end
	elseif sliderFrame:GetLeft() < 0 then
		level.lastDirection = "RIGHT"
		dirty = true
	end
	if level.lastVDirection == "DOWN" then
		if sliderFrame:GetBottom() < 0 then
			level.lastVDirection = "UP"
			dirty = true
		end
	elseif sliderFrame:GetTop() > GetScreenWidth() then
		level.lastVDirection = "DOWN"
		dirty = true
	end
	if dirty then
		sliderFrame:ClearAllPoints()
		if level.lastDirection == "RIGHT" then
			if level.lastVDirection == "DOWN" then
				sliderFrame:SetPoint("TOPLEFT", parent, "TOPRIGHT", 5, 10)
			else
				sliderFrame:SetPoint("BOTTOMLEFT", parent, "BOTTOMRIGHT", 5, -10)
			end
		else
			if level.lastVDirection == "DOWN" then
				sliderFrame:SetPoint("TOPRIGHT", parent, "TOPLEFT", -5, 10)
			else
				sliderFrame:SetPoint("BOTTOMRIGHT", parent, "BOTTOMLEFT", -5, -10)
			end
		end
	end
	local left, bottom = sliderFrame:GetLeft(), sliderFrame:GetBottom()
	sliderFrame:ClearAllPoints()
	sliderFrame:SetPoint("BOTTOMLEFT", UIParent, "BOTTOMLEFT", left, bottom)
	if mod(level.num, 5) == 0 then
		local left, bottom = level:GetLeft(), level:GetBottom()
		level:ClearAllPoints()
		level:SetPoint("BOTTOMLEFT", UIParent, "BOTTOMLEFT", left, bottom)
	end
end

function OpenEditBox(self, parent)
	if not editBoxFrame then
		editBoxFrame = CreateFrame("Frame", nil, UIParent)
		editBoxFrame:SetWidth(200)
		editBoxFrame:SetHeight(40)
		editBoxFrame:SetBackdrop(tmp(
			'bgFile', "Interface\\Tooltips\\UI-Tooltip-Background",
			'edgeFile', "Interface\\Tooltips\\UI-Tooltip-Border",
			'tile', true,
			'insets', tmp2(
				'left', 5,
				'right', 5,
				'top', 5,
				'bottom', 5
			),
			'tileSize', 16,
			'edgeSize', 16
		))
		editBoxFrame:SetFrameStrata("FULLSCREEN_DIALOG")
		if editBoxFrame.SetTopLevel then
			editBoxFrame:SetTopLevel(true)
		end
		editBoxFrame:SetBackdropBorderColor(TOOLTIP_DEFAULT_COLOR.r, TOOLTIP_DEFAULT_COLOR.g, TOOLTIP_DEFAULT_COLOR.b)
		editBoxFrame:SetBackdropColor(TOOLTIP_DEFAULT_BACKGROUND_COLOR.r, TOOLTIP_DEFAULT_BACKGROUND_COLOR.g, TOOLTIP_DEFAULT_BACKGROUND_COLOR.b)
		editBoxFrame:EnableMouse(true)
		editBoxFrame:Hide()
		editBoxFrame:SetPoint("CENTER", UIParent, "CENTER")
		
		local editBox = CreateFrame("EditBox", nil, editBoxFrame)
		editBoxFrame.editBox = editBox
		editBox:SetFontObject(ChatFontNormal)
		editBox:SetWidth(160)
		editBox:SetHeight(13)
		editBox:SetPoint("CENTER", editBoxFrame, "CENTER", 0, 0)
		
		local left = editBox:CreateTexture(nil, "BACKGROUND")
		left:SetTexture("Interface\\ChatFrame\\UI-ChatInputBorder-Left")
		left:SetTexCoord(0, 100 / 256, 0, 1)
		left:SetWidth(100)
		left:SetHeight(32)
		left:SetPoint("LEFT", editBox, "LEFT", -10, 0)
		local right = editBox:CreateTexture(nil, "BACKGROUND")
		right:SetTexture("Interface\\ChatFrame\\UI-ChatInputBorder-Right")
		right:SetTexCoord(156/256, 1, 0, 1)
		right:SetWidth(100)
		right:SetHeight(32)
		right:SetPoint("RIGHT", editBox, "RIGHT", 10, 0)
		
		editBox:SetScript("OnEnterPressed", function()
			if editBoxFrame.parent and editBoxFrame.parent.editBoxValidateFunc then
				local a1,a2,a3,a4 = editBoxFrame.parent.editBoxValidateArg1, editBoxFrame.parent.editBoxValidateArg2, editBoxFrame.parent.editBoxValidateArg3, editBoxFrame.parent.editBoxValidateArg4
				
				local result
				if a1 == nil then
					result = editBoxFrame.parent.editBoxValidateFunc(editBox:GetText() or "")
				elseif a2 == nil then
					result = editBoxFrame.parent.editBoxValidateFunc(a1, editBox:GetText() or "")
				elseif a3 == nil then
					result = editBoxFrame.parent.editBoxValidateFunc(a1, a2, editBox:GetText() or "")
				elseif a4 == nil then
					result = editBoxFrame.parent.editBoxValidateFunc(a1, a2, a3, editBox:GetText() or "")
				else
					result = editBoxFrame.parent.editBoxValidateFunc(a1, a2, a3, a4, editBox:GetText() or "")
				end
				if not result then
					message("Validation error: [" .. tostring(text) .. "]")
					return
				end
			end
			if editBoxFrame.parent and editBoxFrame.parent.editBoxFunc then
				local a1,a2,a3,a4 = editBoxFrame.parent.editBoxArg1, editBoxFrame.parent.editBoxArg2, editBoxFrame.parent.editBoxArg3, editBoxFrame.parent.editBoxArg4
				if a1 == nil then
					editBoxFrame.parent.editBoxFunc(editBox:GetText() or "")
				elseif a2 == nil then
					editBoxFrame.parent.editBoxFunc(a1, editBox:GetText() or "")
				elseif a3 == nil then
					editBoxFrame.parent.editBoxFunc(a1, a2, editBox:GetText() or "")
				elseif a4 == nil then
					editBoxFrame.parent.editBoxFunc(a1, a2, a3, editBox:GetText() or "")
				else
					editBoxFrame.parent.editBoxFunc(a1, a2, a3, a4, editBox:GetText() or "")
				end
			end
			self:Close(editBoxFrame.level)
			for i = 1, editBoxFrame.level - 1 do
				Refresh(self, levels[i])
			end
		end)
		editBox:SetScript("OnEscapePressed", function()
			self:Close(editBoxFrame.level)
		end)
		local changing = false
		local skipNext = false
		
		function editBox:SpecialSetText(text)
			local oldText = editBox:GetText() or ""
			if not text then
				text = ""
			end
			if text ~= oldText then
				changing = true
				self:SetText(text)
				changing = false
				skipNext = true
			end
		end
		
		editBox:SetScript("OnTextChanged", function()
			if skipNext then
				skipNext = false
			elseif not changing and editBoxFrame.parent and editBoxFrame.parent.editBoxChangeFunc then
				local a1,a2,a3,a4 = editBoxFrame.parent.editBoxChangeArg1, editBoxFrame.parent.editBoxChangeArg2, editBoxFrame.parent.editBoxChangeArg3, editBoxFrame.parent.editBoxChangeArg4
				local text
				if a1 == nil then
					text = editBoxFrame.parent.editBoxChangeFunc(editBox:GetText() or "")
				elseif a2 == nil then
					text = editBoxFrame.parent.editBoxChangeFunc(a1, editBox:GetText() or "")
				elseif a3 == nil then
					text = editBoxFrame.parent.editBoxChangeFunc(a1, a2, editBox:GetText() or "")
				elseif a4 == nil then
					text = editBoxFrame.parent.editBoxChangeFunc(a1, a2, a3, editBox:GetText() or "")
				else
					text = editBoxFrame.parent.editBoxChangeFunc(a1, a2, a3, a4, editBox:GetText() or "")
				end
				if text then
					editBox:SpecialSetText(text)
				end
			end
		end)
		editBoxFrame:SetScript("OnEnter", function()
			StopCounting(self, editBoxFrame.level)
		end)
		editBoxFrame:SetScript("OnLeave", function()
			StartCounting(self, editBoxFrame.level)
		end)
		editBox:SetScript("OnEnter", function()
			StopCounting(self, editBoxFrame.level)
		end)
		editBox:SetScript("OnLeave", function()
			StartCounting(self, editBoxFrame.level)
		end)
	end
	editBoxFrame.parent = parent
	editBoxFrame.level = parent.level.num + 1
	editBoxFrame.parentValue = parent.level.value
	editBoxFrame:SetFrameLevel(parent.level:GetFrameLevel() + 3)
	editBoxFrame.editBox:SetFrameLevel(editBoxFrame:GetFrameLevel() + 1)
	editBoxFrame.editBox:SpecialSetText(parent.editBoxText)
	
	if parent.arrow then
--		parent.arrow:SetVertexColor(0.2, 0.6, 0)
		parent.arrow:SetHeight(24)
		parent.arrow:SetWidth(24)
	end
	
	local level = parent.level
	editBoxFrame:Show()
	editBoxFrame:ClearAllPoints()
	if level.lastDirection == "RIGHT" then
		if level.lastVDirection == "DOWN" then
			editBoxFrame:SetPoint("TOPLEFT", parent, "TOPRIGHT", 5, 10)
		else
			editBoxFrame:SetPoint("BOTTOMLEFT", parent, "BOTTOMRIGHT", 5, -10)
		end
	else
		if level.lastVDirection == "DOWN" then
			editBoxFrame:SetPoint("TOPRIGHT", parent, "TOPLEFT", -5, 10)
		else
			editBoxFrame:SetPoint("BOTTOMRIGHT", parent, "BOTTOMLEFT", -5, -10)
		end
	end
	local dirty
	if level.lastDirection == "RIGHT" then
		if editBoxFrame:GetRight() > GetScreenWidth() then
			level.lastDirection = "LEFT"
			dirty = true
		end
	elseif editBoxFrame:GetLeft() < 0 then
		level.lastDirection = "RIGHT"
		dirty = true
	end
	if level.lastVDirection == "DOWN" then
		if editBoxFrame:GetBottom() < 0 then
			level.lastVDirection = "UP"
			dirty = true
		end
	elseif editBoxFrame:GetTop() > GetScreenWidth() then
		level.lastVDirection = "DOWN"
		dirty = true
	end
	if dirty then
		editBoxFrame:ClearAllPoints()
		if level.lastDirection == "RIGHT" then
			if level.lastVDirection == "DOWN" then
				editBoxFrame:SetPoint("TOPLEFT", parent, "TOPRIGHT", 5, 10)
			else
				editBoxFrame:SetPoint("BOTTOMLEFT", parent, "BOTTOMRIGHT", 5, -10)
			end
		else
			if level.lastVDirection == "DOWN" then
				editBoxFrame:SetPoint("TOPRIGHT", parent, "TOPLEFT", -5, 10)
			else
				editBoxFrame:SetPoint("BOTTOMRIGHT", parent, "BOTTOMLEFT", -5, -10)
			end
		end
	end
	local left, bottom = editBoxFrame:GetLeft(), editBoxFrame:GetBottom()
	editBoxFrame:ClearAllPoints()
	editBoxFrame:SetPoint("BOTTOMLEFT", UIParent, "BOTTOMLEFT", left, bottom)
	if mod(level.num, 5) == 0 then
		local left, bottom = level:GetLeft(), level:GetBottom()
		level:ClearAllPoints()
		level:SetPoint("BOTTOMLEFT", UIParent, "BOTTOMLEFT", left, bottom)
	end
end

function Dewdrop:IsOpen(parent)
	self:argCheck(parent, 2, "table", "nil")
	return levels[1] and levels[1]:IsShown() and (not parent or parent == levels[1].parent or parent == levels[1]:GetParent())
end

function Dewdrop:GetOpenedParent()
	return (levels[1] and levels[1]:IsShown()) and (levels[1].parent or levels[1]:GetParent())
end

function Open(self, parent, func, level, value, point, relativePoint, cursorX, cursorY)
	self:Close(level)
	if DewdropLib then
		local d = DewdropLib:GetInstance('1.0')
		local ret, val = pcall(d, IsOpen, d)
		if ret and val then
			DewdropLib:GetInstance('1.0'):Close()
		end
	end
	parent:GetCenter()
	local frame = AcquireLevel(self, level)
	if level == 1 then
		frame.lastDirection = "RIGHT"
		frame.lastVDirection = "DOWN"
	else
		frame.lastDirection = levels[level - 1].lastDirection
		frame.lastVDirection = levels[level - 1].lastVDirection
	end
	frame:SetFrameStrata("FULLSCREEN_DIALOG")
	frame:ClearAllPoints()
	frame.parent = parent
	frame:SetPoint("LEFT", UIParent, "RIGHT", 10000, 0)
	frame:Show()
	if level == 1 then
		baseFunc = func
	end
	levels[level].value = value
--	levels[level].parentText = parent.text and parent.text:GetText() or nil
--	levels[level].parentTooltipTitle = parent.tooltipTitle
--	levels[level].parentTooltipText = parent.tooltipText
--	levels[level].parentTooltipFunc = parent.tooltipFunc
	if parent.arrow then
--		parent.arrow:SetVertexColor(0.2, 0.6, 0)
		parent.arrow:SetHeight(24)
		parent.arrow:SetWidth(24)
	end
	relativePoint = relativePoint or point
	Refresh(self, levels[level])
	if point or (cursorX and cursorY) then
		frame:ClearAllPoints()
		if cursorX and cursorY then
			local curX, curY = GetScaledCursorPosition()
			if curY < GetScreenHeight() / 2 then
				point, relativePoint = "BOTTOM", "BOTTOM"
			else
				point, relativePoint = "TOP", "TOP"
			end
			if curX < GetScreenWidth() / 2 then
				point, relativePoint = point .. "LEFT", relativePoint .. "RIGHT"
			else
				point, relativePoint = point .. "RIGHT", relativePoint .. "LEFT"
			end
		end
		frame:SetPoint(point, parent, relativePoint)
		if cursorX and cursorY then
			local left = frame:GetLeft()
			local width = frame:GetWidth()
			local bottom = frame:GetBottom()
			local height = frame:GetHeight()
			local curX, curY = GetScaledCursorPosition()
			frame:ClearAllPoints()
			relativePoint = relativePoint or point
			if point == "BOTTOM" or point == "TOP" then
				if curX < GetScreenWidth() / 2 then
					point = point .. "LEFT"
				else
					point = point .. "RIGHT"
				end
			elseif point == "CENTER" then
				if curX < GetScreenWidth() / 2 then
					point = "LEFT"
				else
					point = "RIGHT"
				end
			end
			local xOffset, yOffset = 0, 0
			if curY > GetScreenHeight() / 2 then
				yOffset = -height
			end
			if curX > GetScreenWidth() / 2 then
				xOffset = -width
			end
			frame:SetPoint(point, parent, relativePoint, curX - left + xOffset, curY - bottom + yOffset)
			if level == 1 then
				frame.lastDirection = "RIGHT"
			end
		elseif cursorX then
			local left = frame:GetLeft()
			local width = frame:GetWidth()
			local curX, curY = GetScaledCursorPosition()
			frame:ClearAllPoints()
			relativePoint = relativePoint or point
			if point == "BOTTOM" or point == "TOP" then
				if curX < GetScreenWidth() / 2 then
					point = point .. "LEFT"
				else
					point = point .. "RIGHT"
				end
			elseif point == "CENTER" then
				if curX < GetScreenWidth() / 2 then
					point = "LEFT"
				else
					point = "RIGHT"
				end
			end
			frame:SetPoint(point, parent, relativePoint, curX - left - width / 2, 0)
			if level == 1 then
				frame.lastDirection = "RIGHT"
			end
		elseif cursorY then
			local bottom = frame:GetBottom()
			local height = frame:GetHeight()
			local curX, curY = GetScaledCursorPosition()
			frame:ClearAllPoints()
			relativePoint = relativePoint or point
			if point == "LEFT" or point == "RIGHT" then
				if curX < GetScreenHeight() / 2 then
					point = point .. "BOTTOM"
				else
					point = point .. "TOP"
				end
			elseif point == "CENTER" then
				if curX < GetScreenHeight() / 2 then
					point = "BOTTOM"
				else
					point = "TOP"
				end
			end
			frame:SetPoint(point, parent, relativePoint, 0, curY - bottom - height / 2)
			if level == 1 then
				frame.lastDirection = "DOWN"
			end
		end
		if (strsub(point, 1, 3) ~= strsub(relativePoint, 1, 3)) then
			if frame:GetBottom() < 0 then
				local point, parent, relativePoint, x, y = frame:GetPoint(1)
				local change = GetScreenHeight() - frame:GetTop()
				local otherChange = -frame:GetBottom()
				if otherChange < change then
					change = otherChange
				end
				frame:SetPoint(point, parent, relativePoint, x, y + change)
			elseif frame:GetTop() > GetScreenHeight() then
				local point, parent, relativePoint, x, y = frame:GetPoint(1)
				local change = GetScreenHeight() - frame:GetTop()
				local otherChange = -frame:GetBottom()
				if otherChange < change then
					change = otherChange
				end
				frame:SetPoint(point, parent, relativePoint, x, y + change)
			end
		end
	end
	CheckDualMonitor(self, frame)
	StartCounting(self, level)
end

function Dewdrop:IsRegistered(parent)
	self:argCheck(parent, 2, "table")
	return not not self.registry[parent]
end

function Dewdrop:Register(parent, k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20)
	self:argCheck(parent, 2, "table")
	if self.registry[parent] then
		self:Unregister(parent)
	end
	local info = new(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20)
	if type(info.children) == "table" then
		local err, position = validateOptions(info.children)
		
		if err then
			if position then
				Dewdrop:error(position .. ": " .. err)
			else
				Dewdrop:error(err)
			end
		end
	end
	self.registry[parent] = info
	if not info.dontHook and not self.onceRegistered[parent] then
		if parent:HasScript("OnMouseUp") then
			local script = parent:GetScript("OnMouseUp")
			parent:SetScript("OnMouseUp", function()
				if script then
					script()
				end
				if arg1 == "RightButton" and self.registry[parent] then
					if self:IsOpen(parent) then
						self:Close()
					else
						self:Open(parent)
					end
				end
			end)
		end
		if parent:HasScript("OnMouseDown") then
			local script = parent:GetScript("OnMouseDown")
			parent:SetScript("OnMouseDown", function()
				if script then
					script()
				end
				if self.registry[parent] then
					self:Close()
				end
			end)
		end
	end
	self.onceRegistered[parent] = true
end

function Dewdrop:Unregister(parent)
	self:argCheck(parent, 2, "table")
	self.registry[parent] = nil
end

function Dewdrop:Open(parent, k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20)
	self:argCheck(parent, 2, "table")
	local info
	if type(k1) == "table" and k1[0] and k1.IsFrameType and self.registry[k1] then
		info = tmp()
		for k,v in pairs(self.registry[k1]) do
			info[k] = v
		end
	else
		info = tmp(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20)
		if self.registry[parent] then
			for k,v in pairs(self.registry[parent]) do
				if info[k] == nil then
					info[k] = v
				end
			end
		end
	end
	local point = info.point
	local relativePoint = info.relativePoint
	local cursorX = info.cursorX
	local cursorY = info.cursorY
	if type(point) == "function" then
		local b
		point, b = point(parent)
		if b then
			relativePoint = b
		end
	end
	if type(relativePoint) == "function" then
		relativePoint = relativePoint(parent)
	end
	Open(self, parent, info.children, 1, nil, point, relativePoint, cursorX, cursorY)
end

function Clear(self, level)
	if level then
		if level.buttons then
			for i = table.getn(level.buttons), 1, -1 do
				ReleaseButton(self, level, i)
			end
		end
	end
end

function Dewdrop:Close(level)
	if DropDownList1:IsShown() then
		DropDownList1:Hide()
	end
	if DewdropLib then
		local d = DewdropLib:GetInstance('1.0')
		local ret, val = pcall(d, IsOpen, d)
		if ret and val then
			DewdropLib:GetInstance('1.0'):Close()
		end
	end
	self:argCheck(level, 2, "number", "nil")
	if not level then
		level = 1
	end
	if level == 1 and levels[level] then
		levels[level].parented = false
	end
	if level > 1 and levels[level-1].buttons then
		local buttons = levels[level-1].buttons
		for _,button in ipairs(buttons) do
			button.arrow:SetWidth(16)
			button.arrow:SetHeight(16)
--			button.arrow:SetVertexColor(1, 1, 1)
		end
	end
	if sliderFrame and sliderFrame.level >= level then
		sliderFrame:Hide()
	end
	if editBoxFrame and editBoxFrame.level >= level then
		editBoxFrame:Hide()
	end
	for i = level, table.getn(levels) do
		Clear(self, levels[level])
		levels[level]:Hide()
		levels[i]:ClearAllPoints()
		levels[i]:SetPoint("CENTER", UIParent, "CENTER")
	end
end

function Dewdrop:AddLine(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20)
	local info = tmp(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20)
	local level = info.level or currentLevel
	info.level = nil
	local button = AcquireButton(self, level)
	if not next(info) then
		info.disabled = true
	end
	button.disabled = info.isTitle or info.notClickable or info.disabled
	button.isTitle = info.isTitle
	button.notClickable = info.notClickable
	if button.isTitle then
		button.text:SetFontObject(GameFontNormalSmall)
	elseif button.notClickable then
		button.text:SetFontObject(GameFontHighlightSmall)
	elseif button.disabled then
		button.text:SetFontObject(GameFontDisableSmall)
	else
		button.text:SetFontObject(GameFontHighlightSmall)
	end
	if info.disabled then
		button.arrow:SetDesaturated(true)
		button.check:SetDesaturated(true)
	else
		button.arrow:SetDesaturated(false)
		button.check:SetDesaturated(false)
	end
	if info.textR and info.textG and info.textB then
		button.textR = info.textR
		button.textG = info.textG
		button.textB = info.textB
		button.text:SetTextColor(button.textR, button.textG, button.textB)
	else
		button.text:SetTextColor(button.text:GetFontObject():GetTextColor())
	end
	button.notCheckable = info.notCheckable
	button.text:SetPoint("LEFT", button, "LEFT", button.notCheckable and 0 or 24, 0)
	button.checked = not info.notCheckable and info.checked
	button.isRadio = not info.notCheckable and info.isRadio
	if info.isRadio then
		button.check:Show()
		button.check:SetTexture(info.checkIcon or "Interface\\Buttons\\UI-RadioButton")
		if button.checked then
			button.check:SetTexCoord(0.25, 0.5, 0, 1)
			button.check:SetVertexColor(1, 1, 1, 1)
		else
			button.check:SetTexCoord(0, 0.25, 0, 1)
			button.check:SetVertexColor(1, 1, 1, 0.5)
		end
		button.radioHighlight:SetTexture(info.checkIcon or "Interface\\Buttons\\UI-RadioButton")
		button.check:SetWidth(16)
		button.check:SetHeight(16)
	elseif info.icon then 
		button.check:Show()
		button.check:SetTexture(info.icon)
		if info.iconWidth and info.iconHeight then
			button.check:SetWidth(info.iconWidth)
			button.check:SetHeight(info.iconHeight)
		else
			button.check:SetWidth(16)
			button.check:SetHeight(16)
		end
		button.check:SetVertexColor(1, 1, 1, 1)
		button.check:SetTexCoord(0, 1, 0, 1)
	else
		if button.checked then
			if info.checkIcon then
				button.check:SetWidth(16)
				button.check:SetHeight(16)
				button.check:SetTexture(info.checkIcon)
				if string.sub(info.checkIcon, 1, 16) == "Interface\\Icons\\" then
					button.check:SetTexCoord(0.05, 0.95, 0.05, 0.95)
				else
					button.check:SetTexCoord(0, 1, 0, 1)
				end
			else
				button.check:SetWidth(24)
				button.check:SetHeight(24)
				button.check:SetTexture("Interface\\Buttons\\UI-CheckBox-Check")
				button.check:SetTexCoord(0, 1, 0, 1)
			end
			button.check:SetVertexColor(1, 1, 1, 1)
		else
			button.check:SetVertexColor(1, 1, 1, 0)
		end
	end
	if not button.disabled then
		button.func = info.func
	end
	button.hasColorSwatch = info.hasColorSwatch
	if button.hasColorSwatch then
		button.colorSwatch:Show()
		button.colorSwatch.texture:Show()
		button.r = info.r or 1
		button.g = info.g or 1
		button.b = info.b or 1
		button.colorSwatch.texture:SetTexture(button.r, button.g, button.b)
		button.checked = false
		button.func = nil
		button.colorFunc = info.colorFunc
		button.colorArg1 = info.colorArg1
		button.colorArg2 = info.colorArg2
		button.colorArg3 = info.colorArg3
		button.colorArg4 = info.colorArg4
		button.hasOpacity = info.hasOpacity
		button.opacity = info.opacity or 1
	else
		button.colorSwatch:Hide()
		button.colorSwatch.texture:Hide()
	end
	button.hasArrow = not button.hasColorSwatch and (info.value or info.hasSlider or info.hasEditBox) and info.hasArrow
	if button.hasArrow then
		button.arrow:SetAlpha(1)
		if info.hasSlider then
			button.hasSlider = true
			button.sliderMin = info.sliderMin or 0
			button.sliderMax = info.sliderMax or 1
			button.sliderStep = info.sliderStep or 0
			button.sliderIsPercent = info.sliderIsPercent and true or false
			button.sliderMinText = info.sliderMinText or button.sliderIsPercent and string.format("%.0f%%", button.sliderMin * 100) or button.sliderMin
			button.sliderMaxText = info.sliderMaxText or button.sliderIsPercent and string.format("%.0f%%", button.sliderMax * 100) or button.sliderMax
			button.sliderFunc = info.sliderFunc
			button.sliderValue = info.sliderValue
			button.sliderArg1 = info.sliderArg1
			button.sliderArg2 = info.sliderArg2
			button.sliderArg3 = info.sliderArg3
			button.sliderArg4 = info.sliderArg4
		elseif info.hasEditBox then
			button.hasEditBox = true
			button.editBoxText = info.editBoxText or ""
			button.editBoxFunc = info.editBoxFunc
			button.editBoxArg1 = info.editBoxArg1
			button.editBoxArg2 = info.editBoxArg2
			button.editBoxArg3 = info.editBoxArg3
			button.editBoxArg4 = info.editBoxArg4
			button.editBoxChangeFunc = info.editBoxChangeFunc
			button.editBoxChangeArg1 = info.editBoxChangeArg1
			button.editBoxChangeArg2 = info.editBoxChangeArg2
			button.editBoxChangeArg3 = info.editBoxChangeArg3
			button.editBoxChangeArg4 = info.editBoxChangeArg4
			button.editBoxValidateFunc = info.editBoxValidateFunc
			button.editBoxValidateArg1 = info.editBoxValidateArg1
			button.editBoxValidateArg2 = info.editBoxValidateArg2
			button.editBoxValidateArg3 = info.editBoxValidateArg3
			button.editBoxValidateArg4 = info.editBoxValidateArg4
		else
			button.value = info.value
		end
	else
		button.arrow:SetAlpha(0)
	end
	button.arg1 = info.arg1
	button.arg2 = info.arg2
	button.arg3 = info.arg3
	button.arg4 = info.arg4
	button.closeWhenClicked = info.closeWhenClicked
	button.textHeight = info.textHeight or UIDROPDOWNMENU_DEFAULT_TEXT_HEIGHT or 10
	local font,_ = button.text:GetFont()
	button.text:SetFont(STANDARD_TEXT_FONT or "Fonts\\FRIZQT__.TTF", button.textHeight)
	button:SetHeight(button.textHeight + 6)
	button.text:SetPoint("RIGHT", button.arrow, (button.hasColorSwatch or button.hasArrow) and "LEFT" or "RIGHT")
	button.text:SetJustifyH(info.justifyH or "LEFT")
	button.text:SetText(info.text)
	button.tooltipTitle = info.tooltipTitle
	button.tooltipText = info.tooltipText
	button.tooltipFunc = info.tooltipFunc
	button.tooltipArg1 = info.tooltipArg1
	button.tooltipArg2 = info.tooltipArg2
	button.tooltipArg3 = info.tooltipArg3
	button.tooltipArg4 = info.tooltipArg4
	if not button.tooltipTitle and not button.tooltipText and not button.tooltipFunc and not info.isTitle then
		button.tooltipTitle = info.text
	end
	if type(button.func) == "string" then
		self:assert(type(button.arg1) == "table", "Cannot call method " .. button.func .. " on a non-table")
		self:assert(type(button.arg1[button.func]) == "function", "Method " .. button.func .. " nonexistant.")
	end
end

function Dewdrop:InjectAceOptionsTable(handler, options)
	self:argCheck(handler, 2, "table")
	self:argCheck(options, 3, "table")
	if string.lower(tostring(options.type)) ~= "group" then
		self:error('Cannot inject into options table argument #3 if its type is not "group"')
	end
	if options.handler ~= nil and options.handler ~= handler then
		self:error("Cannot inject into options table argument #3 if it has a different handler than argument #2")
	end
	options.handler = handler
	local class = handler.class
	if not AceLibrary:HasInstance("AceOO-2.0") or not class then
		self:error("Cannot retrieve AceOptions tables from a non-object argument #2")
	end
	while class and class ~= AceLibrary("AceOO-2.0").Class do
		if type(class.GetAceOptionsDataTable) == "function" then
			local t = class:GetAceOptionsDataTable(handler)
			for k,v in pairs(t) do
				if type(options.args) ~= "table" then
					options.args = {}
				end
				if options.args[k] == nil then
					options.args[k] = v
				end
			end
		end
		local mixins = class.mixins
		if mixins then
			for mixin in pairs(mixins) do
				if type(mixin.GetAceOptionsDataTable) == "function" then
					local t = mixin:GetAceOptionsDataTable(handler)
					for k,v in pairs(t) do
						if type(options.args) ~= "table" then
							options.args = {}
						end
						if options.args[k] == nil then
							options.args[k] = v
						end
					end
				end
			end
		end
		class = class.super
	end
	return options
end

local function activate(self, oldLib, oldDeactivate)
	Dewdrop = self
	if oldLib and oldLib.registry then
		self.registry = oldLib.registry
		self.onceRegistered = oldLib.onceRegistered
	else
		self.registry = {}
		self.onceRegistered = {}

		local WorldFrame_OnMouseDown = WorldFrame:GetScript("OnMouseDown")
		local WorldFrame_OnMouseUp = WorldFrame:GetScript("OnMouseUp")
		local oldX, oldY, clickTime
		WorldFrame:SetScript("OnMouseDown", function()
			oldX,oldY = GetCursorPosition()
			clickTime = GetTime()
			if WorldFrame_OnMouseDown then
				WorldFrame_OnMouseDown()
			end
		end)

		WorldFrame:SetScript("OnMouseUp", function()
			local x,y = GetCursorPosition()
			if not oldX or not oldY or not x or not y or not clickTime then
				self:Close()
				if WorldFrame_OnMouseUp then
					WorldFrame_OnMouseUp()
				end
				return
			end
			local d = math.abs(x - oldX) + math.abs(y - oldY)
			if d <= 5 and GetTime() - clickTime < 0.5 then
				self:Close()
			end
			if WorldFrame_OnMouseUp then
				WorldFrame_OnMouseUp()
			end
		end)
		
		if hooksecurefunc then
			hooksecurefunc(DropDownList1, "Show", function()
				if levels[1] and levels[1]:IsVisible() then
					self:Close()
				end
			end)
		else
			local DropDownList1_Show = DropDownList1.Show
			function DropDownList1.Show(DropDownList1)
				if levels[1] and levels[1]:IsVisible() then
					self:Close()
				end
				DropDownList1_Show(DropDownList1)
			end
		end
		
		if hooksecurefunc then
			hooksecurefunc("HideDropDownMenu", function()
				if levels[1] and levels[1]:IsVisible() then
					self:Close()
				end
			end)
		else
			local old_HideDropDownMenu = HideDropDownMenu
			function HideDropDownMenu(num)
				if levels[1] and levels[1]:IsVisible() then
					self:Close()
				end
				old_HideDropDownMenu(num)
			end
		end
		
		if hooksecurefunc then
			hooksecurefunc("CloseDropDownMenus", function()
				if levels[1] and levels[1]:IsVisible() then
					self:Close()
				end
			end)
		else
			local old_CloseDropDownMenus = CloseDropDownMenus
			function CloseDropDownMenus(num)
				if levels[1] and levels[1]:IsVisible() then
					self:Close()
				end
				old_CloseDropDownMenus(num)
			end
		end
	end
	levels = {}
	buttons = {}
	
	if oldDeactivate then
		oldDeactivate(oldLib)
	end
end

AceLibrary:Register(Dewdrop, MAJOR_VERSION, MINOR_VERSION, activate)
```

## lib/FuBarPlugin-2.0/FuBarPlugin-2.0.lua
```lua
--[[
Name: FuBarPlugin-2.0
Revision: $Rev: 15281 $
Author: Cameron Kenneth Knight (ckknight@gmail.com)
Website: http://wiki.wowace.com/index.php/FuBarPlugin-2.0
Documentation: http://wiki.wowace.com/index.php/FuBarPlugin-2.0
SVN: svn://svn.wowace.com/root/branches/FuBar/FuBarPlugin-2.0/FuBarPlugin-2.0/
Description: Plugin for FuBar.
Dependencies: AceLibrary, AceOO-2.0, AceEvent-2.0, Tablet-2.0, Dewdrop-2.0
]]

local MAJOR_VERSION = "FuBarPlugin-2.0"
local MINIMAPCONTAINER_MAJOR_VERSION = "FuBarPlugin-MinimapContainer-2.0"
local MINOR_VERSION = "$Revision: 15281 $"

-- This ensures the code is only executed if the libary doesn't already exist, or is a newer version
if not AceLibrary then error(MAJOR_VERSION .. " requires AceLibrary.") end
if not AceLibrary:IsNewVersion(MAJOR_VERSION, MINOR_VERSION) then return end

if not AceLibrary:HasInstance("AceOO-2.0") then error(MAJOR_VERSION .. " requires AceOO-2.0.") end

local AceEvent = AceLibrary:HasInstance("AceEvent-2.0") and AceLibrary("AceEvent-2.0")
local Tablet = AceLibrary:HasInstance("Tablet-2.0") and AceLibrary("Tablet-2.0")
local Dewdrop = AceLibrary:HasInstance("Dewdrop-2.0") and AceLibrary("Dewdrop-2.0")

local epsilon = 1e-5
local _G = getfenv(0)

local SHOW_ICON = "Show icon"
local SHOW_ICON_DESC = "Show the plugins icon on the panel."
local SHOW_TEXT = "Show text"
local SHOW_TEXT_DESC = "Show the plugins text on the panel."
local SHOW_COLORED_TEXT = "Show colored text"
local SHOW_COLORED_TEXT_DESC = "Allow the plugin to color its text."
local DETACH_TOOLTIP = "Detach tooltip"
local DETACH_TOOLTIP_DESC = "Detach the tooltip from the panel."
local LOCK_TOOLTIP = "Lock tooltip"
local LOCK_TOOLTIP_DESC = "Lock the tooltips position."
local POSITION = "Position"
local POSITION_DESC = "Position the plugin on the panel."
local POSITION_LEFT = "Left"
local POSITION_RIGHT = "Right"
local POSITION_CENTER = "Center"
local ATTACH_TO_MINIMAP = "Attach to minimap"
local ATTACH_TO_MINIMAP_DESC = "Attach the plugin to the minimap instead of the panel."
local HIDE_FUBAR_PLUGIN = "Hide FuBar plugin"
local HIDE_FUBAR_PLUGIN_CMD = "Hidden"
local HIDE_FUBAR_PLUGIN_DESC = "Hide the plugin from the panel."

if GetLocale() == "koKR" then
	SHOW_ICON = "아이콘 표시"
	SHOW_ICON_DESC = "패널에 플러그인 아이콘을 표시합니다."
	SHOW_TEXT = "텍스트 표시"
	SHOW_TEXT_DESC = "페널에 플러그인 텍스트를 표시합니다."
	SHOW_COLORED_TEXT = "색상화된 텍스트 표시"
	SHOW_COLORED_TEXT_DESC = "플러그인의 텍스트 색상을 허용합니다."
	DETACH_TOOLTIP = "툴팁 분리"
	DETACH_TOOLTIP_DESC = "패널에서 툴팁을 분리 합니다."
	LOCK_TOOLTIP = "툴팁 고정"
	LOCK_TOOLTIP_DESC = "툴팁 위치를 고정합니다."
	POSITION = "위치"
	POSITION_DESC = "패널에서 플러그인의 위치를 설정합니다."
	POSITION_LEFT = "왼쪽"
	POSITION_RIGHT = "오른쪽"
	POSITION_CENTER = "가운데"
	ATTACH_TO_MINIMAP = "미니맵에 표시"
	ATTACH_TO_MINIMAP_DESC = "플러그인을 패널 대신 미니맵에 표시합니다."
	HIDE_FUBAR_PLUGIN = "FuBar 플러그인 숨기기"
	HIDE_FUBAR_PLUGIN_CMD = "숨겨짐"
	HIDE_FUBAR_PLUGIN_DESC = "패널에서 플러그인을 숨깁니다."
elseif GetLocale() == "deDE" then
	SHOW_ICON = "Zeige Icon"
	SHOW_ICON_DESC = "Zeige das Plugin-Icon auf der Leiste."
	SHOW_TEXT = "Zeige Text"
	SHOW_TEXT_DESC = "Zeige den Plugin-Text auf der Leiste."
	SHOW_COLORED_TEXT = "Zeige gef\195\164rbten Text"
	SHOW_COLORED_TEXT_DESC = "Dem Plugin erlauben sein Text zu f\195\164rben."
	DETACH_TOOLTIP = "Tooltip l\195\182sen"
	DETACH_TOOLTIP_DESC = "Tooltip von der Leiste l\195\182sen."
	LOCK_TOOLTIP = "Tooltip sperren"
	LOCK_TOOLTIP_DESC = "Tooltip an der Position sperren."
	POSITION = "Position"
	POSITION_DESC = "Positioniert das Plugin auf der Leiste."
	POSITION_LEFT = "Links"
	POSITION_RIGHT = "Rechts"
	POSITION_CENTER = "Mitte"
	ATTACH_TO_MINIMAP = "An der Minimap anbringen"
	ATTACH_TO_MINIMAP_DESC = "Bringt das Plugin an der Minimap anstelle der Leiste an."
	HIDE_FUBAR_PLUGIN = "Versteckt das FuBar Plugin"
	HIDE_FUBAR_PLUGIN_CMD = "Verstecken"
	HIDE_FUBAR_PLUGIN_DESC = "Versteckt das Plugin von der Leiste."
elseif GetLocale() == "frFR" then
	SHOW_ICON = "Afficher l'ic\195\180ne"
	SHOW_ICON_DESC = "Afficher l'ic\195\180ne du plugin sur le panneau."
	SHOW_TEXT = "Afficher le texte"
	SHOW_TEXT_DESC = "Afficher le texte du plugin sur le panneau."
	SHOW_COLORED_TEXT = "Afficher la couleur du texte"
	SHOW_COLORED_TEXT_DESC = "Permet au plugin de colorer le texte."
	DETACH_TOOLTIP = "D\195\169tacher le tooltip"
	DETACH_TOOLTIP_DESC = "Permet de d\195\169tacher le tooltip du panneau."
	LOCK_TOOLTIP = "Bloquer le tooltip"
	LOCK_TOOLTIP_DESC = "Permet de bloquer le tooltip \195\160 sa position actuelle."
	POSITION = "Position"
	POSITION_DESC = "Permet de changer la position du plugin dans le panneau."
	POSITION_LEFT = "Gauche"
	POSITION_RIGHT = "Droite"
	POSITION_CENTER = "Centre"
	ATTACH_TO_MINIMAP = "Attacher \195\160 la minicarte"
	ATTACH_TO_MINIMAP_DESC = "Atteche l'ic\195\180ne du plugin \195\160 la minicarte."
	HIDE_FUBAR_PLUGIN = "Masquer le plugin"
	HIDE_FUBAR_PLUGIN_CMD = "Masqu\195\169"
	HIDE_FUBAR_PLUGIN_DESC = "Permet de masquer compl\195\168tement le plugin du panneau."
elseif GetLocale() == "zhCN" then
	SHOW_ICON = "显示图标"
	SHOW_ICON_DESC = "在面板上显示插件图标."
	SHOW_TEXT = "显示文字"
	SHOW_TEXT_DESC = "在面板上显示文字标题."
	SHOW_COLORED_TEXT = "显示彩色文字"
	SHOW_COLORED_TEXT_DESC = "允许插件显示彩色文字."
	DETACH_TOOLTIP = "独立提示信息"
	DETACH_TOOLTIP_DESC = "从面板上独立提示信息."
	LOCK_TOOLTIP = "锁定提示信息"
	LOCK_TOOLTIP_DESC = "锁定提示信息位置."
	POSITION = "位置"
	POSITION_DESC = "插件在面板上的位置."
	POSITION_LEFT = "居左"
	POSITION_RIGHT = "居右"
	POSITION_CENTER = "居中"
	ATTACH_TO_MINIMAP = "依附在小地图"
	ATTACH_TO_MINIMAP_DESC = "插件图标依附在小地图而不显示在面板上."
	HIDE_FUBAR_PLUGIN = "隐藏FuBar插件"
	HIDE_FUBAR_PLUGIN_CMD = "Hidden"
	HIDE_FUBAR_PLUGIN_DESC = "在面板上隐藏该插件."
elseif GetLocale() == "zhTW" then
	SHOW_ICON = "顯示圖示"
	SHOW_ICON_DESC = "在面板上顯示插件圖示。"
	SHOW_TEXT = "顯示文字"
	SHOW_TEXT_DESC = "在面板上顯示文字標題。"
	SHOW_COLORED_TEXT = "顯示彩色文字"
	SHOW_COLORED_TEXT_DESC = "允許插件顯示彩色文字。"
	DETACH_TOOLTIP = "獨立提示訊息"
	DETACH_TOOLTIP_DESC = "從面板上獨立提示訊息。"
	LOCK_TOOLTIP = "鎖定提示訊息"
	LOCK_TOOLTIP_DESC = "鎖定提示訊息位置。"
	POSITION = "位置"
	POSITION_DESC = "插件在面板上的位置。"
	POSITION_LEFT = "靠左"
	POSITION_RIGHT = "靠右"
	POSITION_CENTER = "置中"
	ATTACH_TO_MINIMAP = "依附在小地圖"
	ATTACH_TO_MINIMAP_DESC = "插件圖標依附在小地圖而不顯示在面板上。"
	HIDE_FUBAR_PLUGIN = "隱藏FuBar插件"
	HIDE_FUBAR_PLUGIN_CMD = "Hidden"
	HIDE_FUBAR_PLUGIN_DESC = "在面板上隱藏該插件."
end

local FuBarPlugin = AceLibrary("AceOO-2.0").Mixin {
													"GetTitle",
													"GetName",
													"GetCategory",
													"SetFontSize",
													"GetFrame",
													"Show",
													"Hide",
													"GetPanel",
													"IsTextColored",
													"ToggleTextColored",
													"IsMinimapAttached",
													"ToggleMinimapAttached",
													"Update",
													"UpdateDisplay",
													"UpdateData",
													"UpdateText",
													"UpdateTooltip",
													"SetIcon",
													"GetIcon",
													"CheckWidth",
													"SetText",
													"GetText",
													"IsIconShown",
													"ToggleIconShown",
													"ShowIcon",
													"HideIcon",
													"IsTextShown",
													"ToggleTextShown",
													"ShowText",
													"HideText",
													"IsTooltipDetached",
													"ToggleTooltipDetached",
													"DetachTooltip",
													"ReattachTooltip",
													"GetDefaultPosition",
													"SetPanel",
													"IsLoadOnDemand",
													"IsDisabled",
													"CreateBasicPluginFrame",
													"CreatePluginChildFrame",
													"OpenMenu",
													"AddImpliedMenuOptions",
												  }

local good = nil
local function CheckFuBar()
	if not good then
		good = FuBar and tonumber(string.sub(FuBar.version, 1, 3)) and tonumber(string.sub(FuBar.version, 1, 3)) >= 2 and true
	end
	return good
end

function FuBarPlugin:GetTitle()
	local name = self.title or self.name
	FuBarPlugin:assert(name, "You must provide self.title or self.name")
	local _,_,title = string.find(name, "FuBar %- (.-)%s*$")
	if not title then
		title = name
	end
	return (string.gsub(string.gsub(title, "|c%x%x%x%x%x%x%x%x", ""), "|r", ""))
end

function FuBarPlugin:GetName()
	return self.name
end

function FuBarPlugin:GetCategory()
	return self.category or "Other"
end

function FuBarPlugin:GetFrame()
	return self.frame
end

function FuBarPlugin:GetPanel()
	return self.panel
end

function FuBarPlugin:IsTextColored()
	return not self.db or not self.db.profile or not self.db.profile.uncolored
end

function FuBarPlugin:ToggleTextColored()
	FuBarPlugin:assert(self.db, "Cannot change text color if self.db is not available. (" .. self:GetTitle() .. ")")
	self.db.profile.uncolored = not self.db.profile.uncolored or nil
	self:UpdateText()
end

function FuBarPlugin:ToggleMinimapAttached()
	if CheckFuBar() and not self.cannotAttachToMinimap then
		local value = self:IsMinimapAttached()
		if value then
			if self.panel then
				self.panel:RemovePlugin(self)
			end
			FuBar:GetPanel(1):AddPlugin(self, nil, self.defaultPosition)
		else
			if self.panel then
				self.panel:RemovePlugin(self)
			end
			AceLibrary(MINIMAPCONTAINER_MAJOR_VERSION):AddPlugin(self)
		end
	end
	Dewdrop:Close()
end

function FuBarPlugin:IsMinimapAttached()
	if not CheckFuBar() then
		return true
	end
	return self.panel == AceLibrary(MINIMAPCONTAINER_MAJOR_VERSION)
end

function FuBarPlugin:Update()
	self:UpdateData()
	self:UpdateText()
	self:UpdateTooltip()
end

function FuBarPlugin:UpdateDisplay()
	self:UpdateText()
	self:UpdateTooltip()
end

function FuBarPlugin:UpdateData()
	if type(self.OnDataUpdate) == "function" then
		if not self:IsDisabled() then
			self:OnDataUpdate()
		end
	end
end

function FuBarPlugin:UpdateText()
	if type(self.OnTextUpdate) == "function" then
		if not self:IsDisabled() then
			self:OnTextUpdate()
		end
	elseif self:IsTextShown() then
		self:SetText(self:GetTitle())
	end
end

function FuBarPlugin:RegisterTablet()
	if not Tablet:IsRegistered(self.frame) then
		if self.db and self.db.profile and not self.db.profile.detachedTooltip then
			self.db.profile.detachedTooltip = {}
		end
		Tablet:Register(self.frame,
			'children', function()
				Tablet:SetTitle(self:GetTitle())
				if type(self.OnTooltipUpdate) == "function" then
					if not self:IsDisabled() then
						self:OnTooltipUpdate()
					end
				end
			end,
			'clickable', self.clickableTooltip,
			'data', CheckFuBar() and FuBar.db.profile.tooltip or self.db and self.db.profile.detachedTooltip or {},
			'detachedData', self.db and self.db.profile.detachedTooltip or {},
			'point', function(frame)
				if frame:GetTop() > GetScreenHeight() / 2 then
					local x = frame:GetCenter()
					if x < GetScreenWidth() / 3 then
						return "TOPLEFT", "BOTTOMLEFT"
					elseif x < GetScreenWidth() * 2 / 3 then
						return "TOP", "BOTTOM"
					else
						return "TOPRIGHT", "BOTTOMRIGHT"
					end
				else
					local x = frame:GetCenter()
					if x < GetScreenWidth() / 3 then
						return "BOTTOMLEFT", "TOPLEFT"
					elseif x < GetScreenWidth() * 2 / 3 then
						return "BOTTOM", "TOP"
					else
						return "BOTTOMRIGHT", "TOPRIGHT"
					end
				end
			end,
			'menu', self.OnMenuRequest and function(level, value, valueN_1, valueN_2, valueN_3, valueN_4)
				if level == 1 then
					local name = tostring(self)
					if not string.find(name, '^table:') then
						name = string.gsub(name, "|c%x%x%x%x%x%x%x%x(.-)|r", "%1")
						Dewdrop:AddLine(
							'text', name,
							'isTitle', true
						)
					end
				end
				if type(self.OnMenuRequest) == "function" then
					self:OnMenuRequest(level, value, true, valueN_1, valueN_2, valueN_3, valueN_4)
				elseif type(self.OnMenuRequest) == "table" then
					Dewdrop:FeedAceOptionsTable(self.OnMenuRequest)
				end
			end,
			'hideWhenEmpty', self.tooltipHiddenWhenEmpty
		)
	end
end

function FuBarPlugin:UpdateTooltip()
	FuBarPlugin.RegisterTablet(self)
	if self:IsMinimapAttached() and not self:IsTooltipDetached() and self.minimapFrame then
		Tablet:Refresh(self.minimapFrame)
	else
		Tablet:Refresh(self.frame)
	end
end

function FuBarPlugin:OnProfileEnable()
	self:Update()
end

function FuBarPlugin:Show(panelId)
	if self.frame:IsShown() or (self.minimapFrame and self.minimapFrame:IsShown()) then
		return
	end
	if panelId ~= false then
		if self.db then
			self.db.profile.hidden = nil
		end
	end
	if self.IsActive and not self:IsActive() then
		self.panelIdTmp = panelId
		self:ToggleActive()
		self.panelIdTmp = nil
		if self.db then
			self.db.profile.disabled = nil
		end
	elseif not self.db or not self.db.profile.hidden then
		if panelId == 0 or not CheckFuBar() then
			AceLibrary(MINIMAPCONTAINER_MAJOR_VERSION):AddPlugin(self)
		else
			FuBar:ShowPlugin(self, panelId or self.panelIdTmp)
		end
		if not self.userDefinedFrame then
			if not self:IsTextShown() then
				self.textFrame:SetText("")
				self.textFrame:SetWidth(epsilon)
				self.textFrame:Hide()
			end
			if not self:IsIconShown() then
				self.iconFrame:SetWidth(epsilon)
				self.iconFrame:Hide()
			end
		end
		self:Update()
	end
end

function FuBarPlugin:Hide(check)
	if not self.frame:IsShown() and (not self.minimapFrame or not self.minimapFrame:IsShown()) then
		return
	end
	if self.hideWithoutStandby and self.db and check ~= false then
		self.db.profile.hidden = true
	end
	if not self.hideWithoutStandby then
		if self.db and not self.overrideTooltip and not self.cannotDetachTooltip and self:IsTooltipDetached() and self.db.profile.detachedTooltip and self.db.profile.detachedTooltip.detached then
			self:ReattachTooltip()
			self.db.profile.detachedTooltip.detached = true
		end
		if self.IsActive and self:IsActive() and self.ToggleActive and (not CheckFuBar() or not FuBar:IsChangingProfile()) then
			self:ToggleActive()
		end
	end
	if self.panel then
		self.panel:RemovePlugin(self)
	end
	self.frame:Hide()
	if self.minimapFrame then
		self.minimapFrame:Hide()
	end

	if Dewdrop:IsOpen(self.frame) or (self.minimapFrame and Dewdrop:IsOpen(self.minimapFrame)) then
		Dewdrop:Close()
	end
end

function FuBarPlugin:SetIcon(path)
	if not path then
		return
	end
	FuBarPlugin:argCheck(path, 2, "string", "boolean")
	FuBarPlugin:assert(self.hasIcon, "Cannot set icon unless self.hasIcon is set. (" .. self:GetTitle() .. ")")
	if not self.iconFrame then
		return
	end
	if type(path) ~= "string" then
		path = format("Interface\\AddOns\\%s\\icon", self.folderName)
	elseif not string.find(path, '^Interface[\\/]') then
		path = format("Interface\\AddOns\\%s\\%s", self.folderName, path)
	end
	if string.sub(path, 1, 16) == "Interface\\Icons\\" then
		self.iconFrame:SetTexCoord(0.05, 0.95, 0.05, 0.95)
	else
		self.iconFrame:SetTexCoord(0, 1, 0, 1)
	end
	self.iconFrame:SetTexture(path)
	if self.minimapIcon then
		if string.sub(path, 1, 16) == "Interface\\Icons\\" then
			self.minimapIcon:SetTexCoord(0.05, 0.95, 0.05, 0.95)
		else
			self.minimapIcon:SetTexCoord(0, 1, 0, 1)
		end
		self.minimapIcon:SetTexture(path)
	end
end

function FuBarPlugin:GetIcon()
	if self.hasIcon then
		return self.iconFrame:GetTexture()
	end
end

function FuBarPlugin:CheckWidth(force)
	FuBarPlugin:argCheck(force, 2, "boolean", "nil")
	if (self.iconFrame and self.iconFrame:IsShown()) or (self.textFrame and self.textFrame:IsShown()) then
		if (self.db and self.db.profile and not self:IsIconShown()) or not self.hasIcon then
			self.iconFrame:SetWidth(epsilon)
		end
		local width
		if not self.hasNoText then
			self.textFrame:SetHeight(0)
			self.textFrame:SetWidth(500)
			width = self.textFrame:GetStringWidth() + 1
			self.textFrame:SetWidth(width)
			self.textFrame:SetHeight(self.textFrame:GetHeight())
		end
		if self.hasNoText or not self.textFrame:IsShown() then
			self.frame:SetWidth(self.iconFrame:GetWidth())
			if self.panel and self.panel:GetPluginSide(self) == "CENTER" then
				self.panel:UpdateCenteredPosition()
			end
		elseif force or not self.textWidth or self.textWidth < width or self.textWidth - 8 > width then
			self.textWidth = width
			self.textFrame:SetWidth(width)
			if self.iconFrame and self.iconFrame:IsShown() then
				self.frame:SetWidth(width + self.iconFrame:GetWidth())
			else
				self.frame:SetWidth(width)
			end
			if self.panel and self.panel:GetPluginSide(self) == "CENTER" then
				self.panel:UpdateCenteredPosition()
			end
		end
	end
end

function FuBarPlugin:SetText(text)
	if not self.textFrame then
		return
	end
	FuBarPlugin:assert(not self.hasNoText, "Cannot set text if self.hasNoText has been set. (" .. self:GetTitle() .. ")")
	FuBarPlugin:argCheck(text, 2, "string", "number")
	if text == "" then
		if self.hasIcon then
			self:ShowIcon()
		else
			text = self:GetTitle()
		end
	end
	if not self:IsTextColored() then
		text = string.gsub(string.gsub(text, "|c%x%x%x%x%x%x%x%x", ""), "|r", "")
	end
	self.textFrame:SetText(text)
	self:CheckWidth()
end

function FuBarPlugin:GetText()
	FuBarPlugin:assert(self.textFrame, "Cannot get text without a self.textFrame (" .. self:GetTitle() .. ")")
	if not self.hasNoText then
		return self.textFrame:GetText() or ""
	end
end

function FuBarPlugin:IsIconShown()
	if not self.hasIcon then
		return false
	elseif self.hasNoText then
		return true
	elseif not self.db then
		return true
	elseif self.db.profile.showIcon == nil then
		return true
	else
		return (self.db.profile.showIcon == 1 or self.db.profile.showIcon == true) and true or false
	end
end

function FuBarPlugin:ToggleIconShown()
	FuBarPlugin:assert(self.iconFrame, "Cannot toggle icon without a self.iconFrame (" .. self:GetTitle() .. ")")
	FuBarPlugin:assert(self.hasIcon, "Cannot show icon unless self.hasIcon is set. (" .. self:GetTitle() .. ")")
	FuBarPlugin:assert(not self.hasNoText, "Cannot hide icon if self.hasNoText is set. (" .. self:GetTitle() .. ")")
	FuBarPlugin:assert(self.textFrame, "Cannot hide icon if self.textFrame is not set. (" .. self:GetTitle() .. ")")
	FuBarPlugin:assert(self.iconFrame, "Cannot hide icon if self.iconFrame is not set. (" .. self:GetTitle() .. ")")
	FuBarPlugin:assert(self.db, "Cannot hide icon if self.db is not available. (" .. self:GetTitle() .. ")")
	local value = not self:IsIconShown()
	self.db.profile.showIcon = value
	if value then
		if not self:IsTextShown() and self.textFrame:IsShown() and self.textFrame:GetText() == self:GetTitle() then
			self.textFrame:Hide()
			self.textFrame:SetText("")
		end
		self.iconFrame:Show()
		self.iconFrame:SetWidth(self.iconFrame:GetHeight())
	else
		if not self.textFrame:IsShown() or not self.textFrame:GetText() then
			self.textFrame:Show()
			self.textFrame:SetText(self:GetTitle())
		end
		self.iconFrame:Hide()
		self.iconFrame:SetWidth(epsilon)
	end
	self:CheckWidth(true)
	return value
end

function FuBarPlugin:ShowIcon()
	if not self:IsIconShown() then
		self:ToggleIconShown()
	end
end

function FuBarPlugin:HideIcon()
	if self:IsIconShown() then
		self:ToggleIconShown()
	end
end

function FuBarPlugin:IsTextShown()
	if self.hasNoText then
		return false
	elseif not self.hasIcon then
		return true
	elseif not self.db then
		return true
	elseif self.db.profile.showText == nil then
		return true
	else
		return (self.db.profile.showText == 1 or self.db.profile.showText == true) and true or false
	end
end

function FuBarPlugin:ToggleTextShown()
	FuBarPlugin:assert(not self.cannotHideText, "Cannot hide text unless self.cannotHideText is unset. (" .. self:GetTitle() .. ")")
	FuBarPlugin:assert(self.hasIcon, "Cannot show text unless self.hasIcon is set. (" .. self:GetTitle() .. ")")
	FuBarPlugin:assert(not self.hasNoText, "Cannot hide text if self.hasNoText is set. (" .. self:GetTitle() .. ")")
	FuBarPlugin:assert(self.textFrame, "Cannot hide text if self.textFrame is not set. (" .. self:GetTitle() .. ")")
	FuBarPlugin:assert(self.iconFrame, "Cannot hide text if self.iconFrame is not set. (" .. self:GetTitle() .. ")")
	FuBarPlugin:assert(self.db, "Cannot hide text if self.db is not available. (" .. self:GetTitle() .. ")")
	local value = not self:IsTextShown()
	self.db.profile.showText = value
	if value then
		self.textFrame:Show()
		self:UpdateText()
	else
		self.textFrame:SetText("")
		self.textFrame:SetWidth(epsilon)
		self.textFrame:Hide()
		if not self:IsIconShown() then
			DropDownList1:Hide()
		end
		self:ShowIcon()
	end
	self:CheckWidth(true)
	return value
end

function FuBarPlugin:ShowText()
	if not self:IsTextShown() then
		self:ToggleTextShown()
	end
end

function FuBarPlugin:HideText()
	if self:IsTextShown() then
		self:ToggleTextShown()
	end
end

function FuBarPlugin:IsTooltipDetached()
	FuBarPlugin.RegisterTablet(self)
	return not Tablet:IsAttached(self.frame)
end

function FuBarPlugin:ToggleTooltipDetached()
	FuBarPlugin.RegisterTablet(self)
	if self:IsTooltipDetached() then
		Tablet:Attach(self.frame)
	else
		Tablet:Detach(self.frame)
	end
end

function FuBarPlugin:DetachTooltip()
	FuBarPlugin.RegisterTablet(self)
	Tablet:Detach(self.frame)
end

function FuBarPlugin:ReattachTooltip()
	FuBarPlugin.RegisterTablet(self)
	Tablet:Attach(self.frame)
end

function FuBarPlugin:GetDefaultPosition()
	return self.defaultPosition or "LEFT"
end

local function IsCorrectPanel(panel)
	if type(panel) ~= "table" then
		return false
	elseif type(panel.AddPlugin) ~= "function" then
		return false
	elseif type(panel.RemovePlugin) ~= "function" then
		return false
	elseif type(panel.GetNumPlugins) ~= "function" then
		return false
	elseif type(panel:GetNumPlugins()) ~= "number" then
		return false
	elseif type(panel.GetPlugin) ~= "function" then
		return false
	elseif type(panel.HasPlugin) ~= "function" then
		return false
	elseif type(panel.GetPluginSide) ~= "function" then
		return false
	end
	return true
end

function FuBarPlugin:SetPanel(panel)
	if panel then
		FuBarPlugin:assert(IsCorrectPanel(panel), "Bad argument #2 to `SetPanel'. Panel does not have the correct API.")
	end
	self.panel = panel
end

function FuBarPlugin:SetFontSize(size)
	FuBarPlugin:assert(not self.userDefinedFrame, "You must provide a SetFontSize(size) method if you provide your own frame.")
	if self.hasIcon then
		FuBarPlugin:assert(self.iconFrame, (self.name and self.name .. ": " or "") .. "No iconFrame found")
		self.iconFrame:SetWidth(size + 3)
		self.iconFrame:SetHeight(size + 3)
	end
	if not self.hasNoText then
		FuBarPlugin:assert(self.textFrame, (self.name and self.name .. ": " or "") .. "No textFrame found")
		local font, _, flags = self.textFrame:GetFont()
		self.textFrame:SetFont(font, size, flags)
	end
	self:CheckWidth()
end

function FuBarPlugin:IsLoadOnDemand()
	return IsAddOnLoadOnDemand(self.folderName)
end

function FuBarPlugin:IsDisabled()
	return self.IsActive and not self:IsActive() or false
end

function FuBarPlugin:OnInstanceInit(target)
	if not AceEvent then
		self:error(MAJOR_VERSION .. " requires AceEvent-2.0.")
	elseif not Tablet then
		self:error(MAJOR_VERSION .. " requires Tablet-2.0.")
	elseif not Dewdrop then
		self:error(MAJOR_VERSION .. " requires Dewdrop-2.0.")
	end
	self.registry[target] = true

	local _,_,folderName = string.find(debugstack(6, 1, 0), "\\AddOns\\(.*)\\")
	target.folderName = folderName
	self.folderNames[target] = folderName
end

function FuBarPlugin:CreateBasicPluginFrame(name)
	local frame = CreateFrame("Button", name, UIParent)
	frame:SetFrameStrata("HIGH")
	frame:SetFrameLevel(7)
	frame:EnableMouse(true)
	frame:EnableMouseWheel(true)
	frame:SetMovable(true)
	frame:SetWidth(150)
	frame:SetHeight(24)
	frame:SetPoint("CENTER", UIParent, "CENTER")

	frame:SetScript("OnClick", function()
		if type(self.OnClick) == "function" then
			self:OnClick(arg1)
		end
	end)
	frame:SetScript("OnDoubleClick", function()
		if type(self.OnDoubleClick) == "function" then
			self:OnDoubleClick(arg1)
		end
	end)
	frame:SetScript("OnMouseDown", function()
		if arg1 == "RightButton" and not IsShiftKeyDown() and not IsControlKeyDown() and not IsAltKeyDown() then
			self:OpenMenu()
			return
		else
			HideDropDownMenu(1)
			if type(self.OnMouseDown) == "function" then
				self:OnMouseDown(arg1)
			end
		end
	end)
	frame:SetScript("OnMouseUp", function()
		if type(self.OnMouseUp) == "function" then
			self:OnMouseUp(arg1)
		end
	end)
	frame:SetScript("OnReceiveDrag", function()
		if type(self.OnReceiveDrag) == "function" then
			self:OnReceiveDrag()
		end
	end)
	return frame
end

function FuBarPlugin:CreatePluginChildFrame(frameType, name, parent)
	FuBarPlugin:assert(self.frame, "You must have self.frame declared in order to add child frames")
	FuBarPlugin:argCheck(frameType, 1, "string")
	local child = CreateFrame(frameType, name, parent)
	if parent then
		child:SetFrameLevel(parent:GetFrameLevel() + 2)
	end
	child:SetScript("OnEnter", function()
		if self.frame:GetScript("OnEnter") then
			self.frame:GetScript("OnEnter")()
		end
	end)
	child:SetScript("OnLeave", function()
		if self.frame:GetScript("OnLeave") then
			self.frame:GetScript("OnLeave")()
		end
	end)
	if child:HasScript("OnClick") then
		child:SetScript("OnClick", function()
			if self.frame:HasScript("OnClick") and self.frame:GetScript("OnClick") then
				self.frame:GetScript("OnClick")()
			end
		end)
	end
	if child:HasScript("OnDoubleClick") then
		child:SetScript("OnDoubleClick", function()
			if self.frame:HasScript("OnDoubleClick") and self.frame:GetScript("OnDoubleClick") then
				self.frame:GetScript("OnDoubleClick")()
			end
		end)
	end
	child:SetScript("OnMouseDown", function()
		if self.frame:GetScript("OnMouseDown") then
			self.frame:GetScript("OnMouseDown")()
		end
	end)
	child:SetScript("OnMouseUp", function()
		if self.frame:GetScript("OnMouseUp") then
			self.frame:GetScript("OnMouseUp")()
		end
	end)
	child:SetScript("OnReceiveDrag", function()
		if self.frame:GetScript("OnReceiveDrag") then
			self.frame:GetScript("OnReceiveDrag")()
		end
	end)
	return child
end

function FuBarPlugin:OpenMenu(frame)
	if not frame then
		frame = self:GetFrame()
	end
	if not frame or not self:GetFrame() or Dewdrop:IsOpen(frame) then
		Dewdrop:Close()
		return
	end
	Tablet:Close()

	if not Dewdrop:IsRegistered(self:GetFrame()) then
		if type(self.OnMenuRequest) == "table" and (not self.OnMenuRequest.handler or self.OnMenuRequest.handler == self) and self.OnMenuRequest.type == "group" then
			Dewdrop:InjectAceOptionsTable(self, self.OnMenuRequest)
			if self.OnMenuRequest.args and CheckFuBar() and not self.independentProfile then
				self.OnMenuRequest.args.profile = nil
			end
		end
		Dewdrop:Register(self:GetFrame(),
			'children', type(self.OnMenuRequest) == "table" and self.OnMenuRequest or function(level, value, valueN_1, valueN_2, valueN_3, valueN_4)
				if level == 1 then
					Dewdrop:AddLine(
						'text', self:GetTitle(),
						'isTitle', true
					)
				end

				if level == 1 then
					if self.OnMenuRequest then
						self:OnMenuRequest(level, value, false, valueN_1, valueN_2, valueN_3, valueN_4)
					end

					if not self.overrideMenu then
						if self.MenuSettings then
							Dewdrop:AddLine()
						end
						self:AddImpliedMenuOptions()
					end
				else
					if not self.overrideMenu and self:AddImpliedMenuOptions() then
					else
						if self.OnMenuRequest then
							self:OnMenuRequest(level, value, false, valueN_1, valueN_2, valueN_3, valueN_4)
						end
					end
				end
				if level == 1 then
					Dewdrop:AddLine(
						'text', CLOSE,
						'func', Dewdrop.Close,
						'arg1', Dewdrop
					)
				end
			end,
			'point', function(frame)
				local x, y = frame:GetCenter()
				local leftRight
				if x < GetScreenWidth() / 2 then
					leftRight = "LEFT"
				else
					leftRight = "RIGHT"
				end
				if y < GetScreenHeight() / 2 then
					return "BOTTOM" .. leftRight, "TOP" .. leftRight
				else
					return "TOP" .. leftRight, "BOTTOM" .. leftRight
				end
			end,
			'dontHook', true
		)
	end
	if frame == self:GetFrame() then
		Dewdrop:Open(self:GetFrame())
	else
		Dewdrop:Open(frame, self:GetFrame())
	end
end

local impliedMenuOptions
function FuBarPlugin:AddImpliedMenuOptions(level)
	FuBarPlugin:argCheck(level, 2, "number", "nil")
	if not impliedMenuOptions then
		impliedMenuOptions = {}
	end
	if not impliedMenuOptions[self] then
		impliedMenuOptions[self] = { type = 'group', args = {} }
		Dewdrop:InjectAceOptionsTable(self, impliedMenuOptions[self])
		if impliedMenuOptions[self].args and CheckFuBar() and not self.independentProfile then
			impliedMenuOptions[self].args.profile = nil
		end
	end
	return Dewdrop:FeedAceOptionsTable(impliedMenuOptions[self], level and level - 1)
end

function FuBarPlugin.OnEmbedInitialize(FuBarPlugin, self)
	if not self.frame then
		local name = "FuBarPlugin" .. self:GetTitle() .. "Frame"
		local frame = _G[name]
		if not frame or not _G[name .. "Text"] or not _G[name .. "Icon"] then
			frame = self:CreateBasicPluginFrame(name)

			local icon = frame:CreateTexture(name .. "Icon", "ARTWORK")
			icon:SetWidth(16)
			icon:SetHeight(16)
			icon:SetPoint("LEFT", frame, "LEFT")

			local text = frame:CreateFontString(name .. "Text", "ARTWORK")
			text:SetWidth(134)
			text:SetHeight(24)
			text:SetPoint("LEFT", icon, "RIGHT", 0, 1)
			text:SetFontObject(GameFontNormal)
		end
		self.frame = frame
		self.textFrame = _G[name .. "Text"]
		self.iconFrame = _G[name .. "Icon"]
	else
		self.userDefinedFrame = true
	end

	self.frame.plugin = self
	self.frame:SetParent(UIParent)
	self.frame:SetPoint("RIGHT", UIParent, "LEFT", -5, 0)
	self.frame:Hide()

	if self.hasIcon then
		self:SetIcon(self.hasIcon)
	end

	if CheckFuBar() then
		FuBar:RegisterPlugin(self)
	end
end

local CheckShow = function(self, panelId)
	if not self.frame:IsShown() and (not self.minimapFrame or not self.minimapFrame:IsShown()) then
		self:Show(panelId)
		Dewdrop:Refresh(2)
	end
end

local recheckPlugins
function FuBarPlugin.OnEmbedEnable(FuBarPlugin, self)
	if not self.userDefinedFrame then
		if self:IsIconShown() then
			self.iconFrame:Show()
		else
			self.iconFrame:Hide()
		end
	end
	self:CheckWidth(true)

	if not self.hideWithoutStandby or not self.db.profile.hidden then
		if FuBarPlugin.enabledPlugins[self] then
			CheckShow(self, self.panelIdTmp)
		else
			FuBarPlugin:ScheduleEvent(CheckShow, 0, self, self.panelIdTmp)
		end
	end
	FuBarPlugin.enabledPlugins[self] = true

	if not self.overrideTooltip and not self.cannotDetachTooltip and self.db and self.db.profile.detachedTooltip and self.db.profile.detachedTooltip.detached then
		FuBarPlugin:ScheduleEvent(self.DetachTooltip, 0, self)
	end

	if self:IsLoadOnDemand() and CheckFuBar() then
		if not FuBar.db.profile.loadOnDemand then
			FuBar.db.profile.loadOnDemand = {}
		end
		if not FuBar.db.profile.loadOnDemand[self.folderName] then
			FuBar.db.profile.loadOnDemand[self.folderName] = {}
		end
		FuBar.db.profile.loadOnDemand[self.folderName].disabled = nil
	end

	if CheckFuBar() and AceLibrary:HasInstance("AceConsole-2.0") then
		if not recheckPlugins then
			local AceConsole = AceLibrary("AceConsole-2.0")
			local AceOO = AceLibrary("AceOO-2.0")
			function recheckPlugins()
				for k,v in pairs(AceConsole.registry) do
					if type(v) == "table" and v.args and AceOO.inherits(v.handler, FuBarPlugin) and not v.independentProfile then
						v.args.profile = nil
					end
				end
			end
		end
		FuBarPlugin:ScheduleEvent(recheckPlugins, 0)
	end
end

function FuBarPlugin.OnEmbedDisable(FuBarPlugin, self)
	self:Hide(false)

	if self:IsLoadOnDemand() and CheckFuBar() then
		if not FuBar.db.profile.loadOnDemand then
			FuBar.db.profile.loadOnDemand = {}
		end
		if not FuBar.db.profile.loadOnDemand[self.folderName] then
			FuBar.db.profile.loadOnDemand[self.folderName] = {}
		end
		FuBar.db.profile.loadOnDemand[self.folderName].disabled = true
	end
end

function FuBarPlugin.OnEmbedProfileEnable(FuBarPlugin, self)
	self:Update()
	if self.db and self.db.profile then
		if not self.db.profile.detachedTooltip then
			self.db.profile.detachedTooltip = {}
		end
		if Tablet.registry[self.frame] then
			Tablet:UpdateDetachedData(self.frame, self.db.profile.detachedTooltip)
		else
			FuBarPlugin.RegisterTablet(self)
		end
	end
end

function FuBarPlugin.GetAceOptionsDataTable(FuBarPlugin, self)
	return {
		icon = {
			type = "toggle",
			name = SHOW_ICON,
			desc = SHOW_ICON_DESC,
			set = "ToggleIconShown",
			get = "IsIconShown",
			hidden = function()
				return not self.hasIcon or self.hasNoText or self:IsDisabled() or self:IsMinimapAttached() or not self.db
			end,
			order = -13.7,
			handler = self,
		},
		text = {
			type = "toggle",
			name = SHOW_TEXT,
			desc = SHOW_TEXT_DESC,
			set = "ToggleTextShown",
			get = "IsTextShown",
			hidden = function()
				return self.cannotHideText or not self.hasIcon or self.hasNoText or self:IsDisabled() or self:IsMinimapAttached() or not self.db
			end,
			order = -13.6,
			handler = self,
		},
		colorText = {
			type = "toggle",
			name = SHOW_COLORED_TEXT,
			desc = SHOW_COLORED_TEXT_DESC,
			set = "ToggleTextColored",
			get = "IsTextColored",
			hidden = function()
				return self.userDefinedFrame or self.hasNoText or self.hasNoColor or self:IsDisabled() or self:IsMinimapAttached() or not self.db
			end,
			order = -13.5,
			handler = self,
		},
		detachTooltip = {
			type = "toggle",
			name = DETACH_TOOLTIP,
			desc = DETACH_TOOLTIP_DESC,
			get = "IsTooltipDetached",
			set = "ToggleTooltipDetached",
			hidden = function()
				return self.overrideTooltip or self.cannotDetachTooltip or self:IsDisabled()
			end,
			order = -13.4,
			handler = self,
		},
		lockTooltip = {
			type = "toggle",
			name = LOCK_TOOLTIP,
			desc = LOCK_TOOLTIP_DESC,
			get = function()
				return Tablet:IsLocked(self.frame)
			end,
			set = function()
				return Tablet:ToggleLocked(self.frame)
			end,
			disabled = function()
				return not self:IsTooltipDetached()
			end,
			hidden = function()
				return self.overrideTooltip or self.cannotDetachTooltip or self:IsDisabled()
			end,
			order = -13.3,
			handler = self,
		},
		position = {
			type = "text",
			name = POSITION,
			desc = POSITION_DESC,
			validate = {
				LEFT = POSITION_LEFT,
				CENTER = POSITION_CENTER,
				RIGHT = POSITION_RIGHT
			},
			get = function()
				return self.panel and self.panel:GetPluginSide(self)
			end,
			set = function(value)
				if self.panel then
					self.panel:SetPluginSide(self, value)
				end
			end,
			hidden = function()
				return self:IsMinimapAttached() or self:IsDisabled() or not self.panel
			end,
			order = -13.2,
			handler = self,
		},
		minimapAttach = {
			type = "toggle",
			name = ATTACH_TO_MINIMAP,
			desc = ATTACH_TO_MINIMAP_DESC,
			get = "IsMinimapAttached",
			set = "ToggleMinimapAttached",
			hidden = function()
				return (self.cannotAttachToMinimap and not self:IsMinimapAttached()) or not CheckFuBar() or self:IsDisabled()
			end,
			order = -13.1,
			handler = self,
		},
		hide = {
			type = "toggle",
			cmdName = HIDE_FUBAR_PLUGIN_CMD,
			guiName = HIDE_FUBAR_PLUGIN,
			desc = HIDE_FUBAR_PLUGIN_DESC,
			get = function()
				return not self.frame:IsShown() and (not self.minimapFrame or not self.minimapFrame:IsShown())
			end,
			set = function()
				if not self.frame:IsShown() and (not self.minimapFrame or not self.minimapFrame:IsShown()) then
					self:Show()
				else
					self:Hide()
				end
			end,
			hidden = function()
				return not self.hideWithoutStandby or self:IsDisabled()
			end,
			order = -13,
			handler = self,
		},
	}
end

local function activate(self, oldLib, oldDeactivate)
	FuBarPlugin = self

	if oldLib then
		self.registry = oldLib.registry
		self.folderNames = oldLib.folderNames
		self.enabledPlugins = oldLib.enabledPlugins
	end

	if not self.registry then
		self.registry = {}
	end
	if not self.folderNames then
		self.folderNames = {}
	end
	if not self.enabledPlugins then
		self.enabledPlugins = {}
	end

	FuBarPlugin.activate(self, oldLib, oldDeactivate)

	if oldDeactivate then
		oldDeactivate(oldLib)
	end
end

local function external(self, major, instance)
	if major == "AceEvent-2.0" then
		AceEvent = instance

		AceEvent:embed(self)
	elseif major == "Tablet-2.0" then
		Tablet = instance
	elseif major == "Dewdrop-2.0" then
		Dewdrop = instance
	end
end

AceLibrary:Register(FuBarPlugin, MAJOR_VERSION, MINOR_VERSION, activate, nil, external)

local MinimapContainer = {}

local IsMinimapSquare
do
	local value
	function IsMinimapSquare()
		if value == nil then
			if not AceEvent or not AceEvent:IsFullyInitialized() then
				return IsAddOnLoaded("CornerMinimap") or IsAddOnLoaded("SquareMinimap") or IsAddOnLoaded("Squeenix")
			else
				value = IsAddOnLoaded("CornerMinimap") or IsAddOnLoaded("SquareMinimap") or IsAddOnLoaded("Squeenix") and true or false
			end
		end
		return value
	end
end

function MinimapContainer:AddPlugin(plugin)
	if CheckFuBar() and FuBar:IsChangingProfile() then
		return
	end
	if plugin.panel ~= nil then
		plugin.panel:RemovePlugin(plugin)
	end
	plugin.panel = self
	if not plugin.minimapFrame then
		local frame = CreateFrame("Button", plugin.frame:GetName() .. "MinimapButton", Minimap)
		plugin.minimapFrame = frame
		AceLibrary(MAJOR_VERSION).RegisterTablet(plugin)
		Tablet:Register(frame, plugin.frame)
		frame.plugin = plugin
		frame:SetWidth(31)
		frame:SetHeight(31)
		frame:SetFrameStrata("BACKGROUND")
		frame:SetFrameLevel(4)
		frame:SetHighlightTexture("Interface\\Minimap\\UI-Minimap-ZoomButton-Highlight")
		local icon = frame:CreateTexture(frame:GetName() .. "Icon", "BACKGROUND")
		plugin.minimapIcon = icon
		local path = plugin:GetIcon() or (plugin.iconFrame and plugin.iconFrame:GetTexture()) or "Interface\\Icons\\INV_Misc_QuestionMark"
		icon:SetTexture(path)
		if string.sub(path, 1, 16) == "Interface\\Icons\\" then
			icon:SetTexCoord(0.05, 0.95, 0.05, 0.95)
		else
			icon:SetTexCoord(0, 1, 0, 1)
		end
		icon:SetWidth(20)
		icon:SetHeight(20)
		icon:SetPoint("TOPLEFT", frame, "TOPLEFT", 7, -5)
		local overlay = frame:CreateTexture(frame:GetName() .. "Overlay","OVERLAY")
overlay:SetTexture("Interface\\Minimap\\MiniMap-TrackingBorder")
		overlay:SetWidth(53)
		overlay:SetHeight(53)
		overlay:SetPoint("TOPLEFT",frame,"TOPLEFT")
		frame:EnableMouse(true)
		frame:RegisterForClicks("LeftButtonUp")
		frame.plugin = plugin
		frame:SetScript("OnClick", function()
			if type(plugin.OnClick) == "function" then
				if not this.dragged then
					plugin:OnClick(arg1)
				end
			end
		end)
		frame:SetScript("OnDoubleClick", function()
			if type(plugin.OnDoubleClick) == "function" then
				plugin:OnDoubleClick(arg1)
			end
		end)
		frame:SetScript("OnReceiveDrag", function()
			if type(plugin.OnReceiveDrag) == "function" then
				if not this.dragged then
					plugin:OnReceiveDrag()
				end
			end
		end)
		frame:SetScript("OnMouseDown", function()
			this.dragged = false
			if arg1 == "LeftButton" and not IsShiftKeyDown() and not IsControlKeyDown() and not IsAltKeyDown() then
				HideDropDownMenu(1)
				if type(plugin.OnMouseDown) == "function" then
					plugin:OnMouseDown(arg1)
				end
			elseif arg1 == "RightButton" and not IsShiftKeyDown() and not IsControlKeyDown() and not IsAltKeyDown() then
				plugin:OpenMenu(frame)
			else
				HideDropDownMenu(1)
				if type(plugin.OnMouseDown) == "function" then
					plugin:OnMouseDown(arg1)
				end
			end
			if plugin.OnClick or plugin.OnMouseDown or plugin.OnMouseUp or plugin.OnDoubleClick then
				if string.sub(this.plugin.minimapIcon:GetTexture(), 1, 16) == "Interface\\Icons\\" then
					plugin.minimapIcon:SetTexCoord(0.14, 0.86, 0.14, 0.86)
				else
					plugin.minimapIcon:SetTexCoord(0.1, 0.9, 0.1, 0.9)
				end
			end
		end)
		frame:SetScript("OnMouseUp", function()
			if not this.dragged and type(plugin.OnMouseUp) == "function" then
				plugin:OnMouseUp(arg1)
			end
			if string.sub(this.plugin.minimapIcon:GetTexture(), 1, 16) == "Interface\\Icons\\" then
				plugin.minimapIcon:SetTexCoord(0.05, 0.95, 0.05, 0.95)
			else
				plugin.minimapIcon:SetTexCoord(0, 1, 0, 1)
			end
		end)
		frame:RegisterForDrag("LeftButton")
		frame:SetScript("OnDragStart", self.OnDragStart)
		frame:SetScript("OnDragStop", self.OnDragStop)
	end
	plugin.frame:Hide()
	plugin.minimapFrame:Show()
	self:ReadjustLocation(plugin)
	table.insert(self.plugins, plugin)
	local exists = false
	return true
end

function MinimapContainer:RemovePlugin(index)
	if CheckFuBar() and FuBar:IsChangingProfile() then
		return
	end
	if type(index) == "table" then
		index = self:IndexOfPlugin(index)
		if not index then
			return
		end
	end
	local t = self.plugins
	local plugin = t[index]
	assert(plugin.panel == self, "Plugin has improper panel field")
	plugin:SetPanel(nil)
	table.remove(t, index)
	return true
end

function MinimapContainer:ReadjustLocation(plugin)
	local frame = plugin.minimapFrame
	if plugin.db and plugin.db.profile.minimapPositionWild then
		frame:SetPoint("CENTER", UIParent, "BOTTOMLEFT", plugin.db.profile.minimapPositionX, plugin.db.profile.minimapPositionY)
	elseif not plugin.db and plugin.minimapPositionWild then
		frame:SetPoint("CENTER", UIParent, "BOTTOMLEFT", plugin.minimapPositionX, plugin.minimapPositionY)
	else
		local position
		if plugin.db then
			position = plugin.db.profile.minimapPosition or plugin.defaultMinimapPosition or math.random(1, 360)
		else
			position = plugin.minimapPosition or plugin.defaultMinimapPosition or math.random(1, 360)
		end
		local angle = math.rad(position or 0)
		local x,y
		if not IsMinimapSquare() then
			x = math.cos(angle) * 80
			y = math.sin(angle) * 80
		else
			x = 110 * math.cos(angle)
			y = 110 * math.sin(angle)
			x = math.max(-82, math.min(x, 84))
			y = math.max(-86, math.min(y, 82))
		end
		frame:SetPoint("CENTER", Minimap, "CENTER", x, y)
	end
end

function MinimapContainer:GetPlugin(index)
	return self.plugins[index]
end

function MinimapContainer:GetNumPlugins()
	return table.getn(self.plugins)
end

function MinimapContainer:IndexOfPlugin(plugin)
	for i,p in ipairs(self.plugins) do
		if p == plugin then
			return i, "MINIMAP"
		end
	end
end

function MinimapContainer:HasPlugin(plugin)
	return self:IndexOfPlugin(plugin) ~= nil
end

function MinimapContainer:GetPluginSide(plugin)
	local index = self:IndexOfPlugin(plugin)
	assert(index, "Plugin not in panel")
	return "MINIMAP"
end

function MinimapContainer.OnDragStart()
	this.dragged = true
	this:LockHighlight()
	this:SetScript("OnUpdate", MinimapContainer.OnUpdate)
	if string.sub(this.plugin.minimapIcon:GetTexture(), 1, 16) == "Interface\\Icons\\" then
		this.plugin.minimapIcon:SetTexCoord(0.05, 0.95, 0.05, 0.95)
	else
		this.plugin.minimapIcon:SetTexCoord(0, 1, 0, 1)
	end
end

function MinimapContainer.OnDragStop()
	this:SetScript("OnUpdate", nil)
	this:UnlockHighlight()
end

function MinimapContainer.OnUpdate()
	if not IsAltKeyDown() then
		local mx, my = Minimap:GetCenter()
		local px, py = GetCursorPosition()
		local scale = UIParent:GetEffectiveScale()
		px, py = px / scale, py / scale
		local position = math.deg(math.atan2(py - my, px - mx))
		if position <= 0 then
			position = position + 360
		elseif position > 360 then
			position = position - 360
		end
		if this.plugin.db then
			this.plugin.db.profile.minimapPosition = position
			this.plugin.db.profile.minimapPositionX = nil
			this.plugin.db.profile.minimapPositionY = nil
			this.plugin.db.profile.minimapPositionWild = nil
		else
			this.plugin.minimapPosition = position
			this.plugin.minimapPositionX = nil
			this.plugin.minimapPositionY = nil
			this.plugin.minimapPositionWild = nil
		end
	else
		local px, py = GetCursorPosition()
		local scale = UIParent:GetEffectiveScale()
		px, py = px / scale, py / scale
		if this.plugin.db then
			this.plugin.db.profile.minimapPositionX = px
			this.plugin.db.profile.minimapPositionY = py
			this.plugin.db.profile.minimapPosition = nil
			this.plugin.db.profile.minimapPositionWild = true
		else
			this.plugin.minimapPositionX = px
			this.plugin.minimapPositionY = py
			this.plugin.minimapPosition = nil
			this.plugin.minimapPositionWild = true
		end
	end
	MinimapContainer:ReadjustLocation(this.plugin)
end

local function activate(self, oldLib, oldDeactivate)
	MinimapContainer = self

	if oldLib then
		self.plugins = oldLib.plugins
	end

	if not self.plugins then
		self.plugins = {}
	end

	if oldDeactivate then
		oldDeactivate(oldLib)
	end
end

AceLibrary:Register(MinimapContainer, MINIMAPCONTAINER_MAJOR_VERSION, MINOR_VERSION, activate)
```

## lib/Tablet-2.0/Tablet-2.0.lua
```lua
--[[
Name: Tablet-2.0
Revision: $Rev: 17999 $
Author(s): ckknight (ckknight@gmail.com)
Website: http://ckknight.wowinterface.com/
Documentation: http://wiki.wowace.com/index.php/Tablet-2.0
SVN: http://svn.wowace.com/root/trunk/TabletLib/Tablet-2.0
Description: A library to provide an efficient, featureful tooltip-style display.
Dependencies: AceLibrary, (optional) Dewdrop-2.0
]]

local MAJOR_VERSION = "Tablet-2.0"
local MINOR_VERSION = "$Revision: 17999 $"

if not AceLibrary then error(MAJOR_VERSION .. " requires AceLibrary") end
if not AceLibrary:IsNewVersion(MAJOR_VERSION, MINOR_VERSION) then return end

local DEBUG = false


local SCROLL_UP = "Scroll up"
local SCROLL_DOWN = "Scroll down"
local HINT = "Hint"
local DETACH = "Detach"
local DETACH_DESC = "Detach the tablet from its source."
local SIZE = "Size"
local SIZE_DESC = "Scale the tablet."
local CLOSE_MENU = "Close menu"
local CLOSE_MENU_DESC = "Close the menu."
local COLOR = "Background color"
local COLOR_DESC = "Set the background color."
local LOCK = "Lock"
local LOCK_DESC = "Lock the tablet in its current position. Alt+Right-click for menu or Alt+drag to drag it when locked."

if GetLocale() == "deDE" then
	SCROLL_UP = "Hochscrollen"
	SCROLL_DOWN = "Runterscrollen"
	HINT = "Hinweis"
	DETACH = "L\195\182sen"
	DETACH_DESC = "L\195\182st den Tooltip aus seiner Verankerung."
	SIZE = "Gr\195\182\195\159e"
	SIZE_DESC = "Gr\195\182\195\159e des Tooltips \195\164ndern."
	CLOSE_MENU = "Menu schlie\195\159en"
	CLOSE_MENU_DESC = "Schlie\195\159t das Menu."
	COLOR = "Hintergrundfarbe"
	COLOR_DESC = "Hintergrundfarbe setzen."
	LOCK = "Sperren"
	LOCK_DESC = "Sperrt die aktuelle Position vom Tooltip. Alt+Rechts-klick f\195\188rs Men\195\188 oder Alt+Verschieben f\195\188rs verschieben wenn es gesperrt ist."
elseif  GetLocale() == "ruRU" then
	SCROLL_UP = "Прокрутка вверх"
	SCROLL_DOWN = "Прокрутка вниз"
	HINT = "Совет"
	DETACH = "Отделить"
	DETACH_DESC = "Отделить планшет от его источника."
	SIZE = "Размер"
	SIZE_DESC = "Масштаб планшета."
	CLOSE_MENU = "Закрыть меню"
	CLOSE_MENU_DESC = "Закрыть меню."
	COLOR = "Цвет фона"
	COLOR_DESC = "Установить цвет фона."
	LOCK = "Зафиксировать"
	LOCK_DESC = "Зафиксировать планшет в его текущем позиции. Alt+ПКМ для меню или Alt+перетаскивание для перетаскивания когда планшет зафиксирован."
elseif GetLocale() == "koKR" then
	SCROLL_UP = "위로 스크롤"
	SCROLL_DOWN = "아래로 스크롤"
	HINT = "힌트"
	DETACH = "분리"
	DETACH_DESC = "테이블을 분리합니다."
	SIZE = "크기"
	SIZE_DESC = "테이블의 크기입니다."
	CLOSE_MENU = "메뉴 닫기"
	CLOSE_MENU_DESC = "메뉴를 닫습니다."
	COLOR = "배경 색상"
	COLOR_DESC = "배경 색상을 설정합니다."
	LOCK = "고정"
	LOCK_DESC = "현재 위치에 테이블을 고정합니다. 알트+우클릭 : 메뉴열기, 알트+드래그 : 고정된것을 드래그합니다."
elseif GetLocale() == "zhCN" then
	SCROLL_UP = "向上翻转"
	SCROLL_DOWN = "向上翻转"
	HINT = "提示"
	DETACH = "分离"
	DETACH_DESC = "分离菜单为独立提示."
	SIZE = "尺寸"
	SIZE_DESC = "缩放菜单显示尺寸."
	CLOSE_MENU = "关闭菜单"
	CLOSE_MENU_DESC = "关闭菜单"
	COLOR = "背景颜色"
	COLOR_DESC = "设置菜单背景颜色."
	LOCK = "锁定"
	LOCK_DESC = "锁定菜单当前位置. alt+右键 将显示选项, alt+拖动 可以移动已锁定的菜单."
elseif GetLocale() == "zhTW" then
	SCROLL_UP = "向上翻轉"
	SCROLL_DOWN = "向上翻轉"
	HINT = "提示"
	DETACH = "分離"
	DETACH_DESC = "分離選單為獨立提示。"
	SIZE = "尺寸"
	SIZE_DESC = "縮放選單顯示尺寸。"
	CLOSE_MENU = "關閉選單"
	CLOSE_MENU_DESC = "關閉選單"
	COLOR = "背景顏色"
	COLOR_DESC = "設置選單背景顏色。"
	LOCK = "鎖定"
	LOCK_DESC = "鎖定選單目前位置. Alt+右鍵 將顯示選項，Alt+拖動 可以移動已鎖定的選單。"
elseif GetLocale() == "frFR" then
	SCROLL_UP = "Parcourir vers le haut"
	SCROLL_DOWN = "Parcourir vers le bas"
	HINT = "Astuce"
	DETACH = "D\195\169tacher"
	DETACH_DESC = "Permet de d\195\169tacher le tableau de sa source."
	SIZE = "Taille"
	SIZE_DESC = "Permet de changer l'\195\169chelle du tableau."
	CLOSE_MENU = "Fermer le menu"
	CLOSE_MENU_DESC = "Ferme ce menu."
	COLOR = "Couleur du fond"
	COLOR_DESC = "Permet de d\195\169finir la couleur du fond."
	LOCK = "Bloquer"
	LOCK_DESC = "Bloque le tableau \195\160 sa position actuelle. Alt+clic-droit pour le menu ou Alt+glisser pour le d\195\169placer quand il est bloqu\195\169."
end

local start = GetTime()
local wrap
local GetProfileInfo
if DEBUG then
	local tree = {}
	local treeMemories = {}
	local treeTimes = {}
	local memories = {}
	local times = {}
	function wrap(value, name)
		if type(value) == "function" then
			local oldFunction = value
			memories[name] = 0
			times[name] = 0
			return function(self, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20, a21, a22, a23, a24, a25, a26, a27, a28, a29, a30, a31, a32, a33, a34, a35, a36, a37, a38, a39, a40, a41, a42, a43, a44, a45, a46, a47, a48, a49, a50, a51, a52, a53, a54, a55, a56, a57, a58, a59, a60)
				local pos = table.getn(tree)
				table.insert(tree, name)
				table.insert(treeMemories, 0)
				table.insert(treeTimes, 0)
				local t, mem = GetTime(), gcinfo()
				local r1, r2, r3, r4, r5, r6, r7, r8 = oldFunction(self, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12, a13, a14, a15, a16, a17, a18, a19, a20, a21, a22, a23, a24, a25, a26, a27, a28, a29, a30, a31, a32, a33, a34, a35, a36, a37, a38, a39, a40, a41, a42, a43, a44, a45, a46, a47, a48, a49, a50, a51, a52, a53, a54, a55, a56, a57, a58, a59, a60)
				mem, t = gcinfo() - mem, GetTime() - t
				if pos > 0 then
					treeMemories[pos] = treeMemories[pos] + mem
					treeTimes[pos] = treeTimes[pos] + t
				end
				local otherMem = table.remove(treeMemories)
				if mem - otherMem > 0 then
					memories[name] = memories[name] + mem - otherMem
				end
				times[name] = times[name] + t - table.remove(treeTimes)
				table.remove(tree)
				return r1, r2, r3, r4, r5, r6, r7, r8
			end
		end
	end

	function GetProfileInfo()
		return GetTime() - start, times, memories
	end
else
	function wrap(value)
		return value
	end
end

local MIN_TOOLTIP_SIZE = 200
local TESTSTRING_EXTRA_WIDTH = 5
local Tablet = {}
local function getsecond(_, value)
	return value
end
local Dewdrop
local sekeys
local CleanCategoryPool
local pool = {}

local lua51 = loadstring("return function(...) return ... end") and true or false
local table_setn = lua51 and function() end or table.setn

local function del(t)
	if t then
		for k in pairs(t) do
			t[k] = nil
		end
		setmetatable(t, nil)
		table_setn(t, 0)
		table.insert(pool, t)
	end
end

local new

local function copy(parent)
	local t
	if table.getn(pool) > 0 then
		t = table.remove(pool)
	else
		t = {}
	end
	if parent then
		for k,v in pairs(parent) do
			t[k] = v
		end
		table_setn(t, table.getn(parent))
		setmetatable(t, getmetatable(parent))
	end
	return t
end

function new(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20, k21, v21, k22, v22, k23, v23, k24, v24, k25, v25, k26, v26, k27, v27, k28, v28, k29, v29, k30, v30)
	local t
	if table.getn(pool) > 0 then
		t = table.remove(pool)
	else
		t = {}
	end
	if k1 then t[k1] = v1
	if k2 then t[k2] = v2
	if k3 then t[k3] = v3
	if k4 then t[k4] = v4
	if k5 then t[k5] = v5
	if k6 then t[k6] = v6
	if k7 then t[k7] = v7
	if k8 then t[k8] = v8
	if k9 then t[k9] = v9
	if k10 then t[k10] = v10
	if k11 then t[k11] = v11
	if k12 then t[k12] = v12
	if k13 then t[k13] = v13
	if k14 then t[k14] = v14
	if k15 then t[k15] = v15
	if k16 then t[k16] = v16
	if k17 then t[k17] = v17
	if k18 then t[k18] = v18
	if k19 then t[k19] = v19
	if k20 then t[k20] = v20
	if k21 then t[k21] = v21
	if k22 then t[k22] = v22
	if k23 then t[k23] = v23
	if k24 then t[k24] = v24
	if k25 then t[k25] = v25
	if k26 then t[k26] = v26
	if k27 then t[k27] = v27
	if k28 then t[k28] = v28
	if k29 then t[k29] = v29
	if k30 then t[k30] = v30
	end end end end end end end end end end end end end end end end end end end end end end end end end end end end end end
	return t
end
local tmp
tmp = setmetatable({}, {__index = function(self, key)
	local t = {}
	tmp[key] = function(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20, k21, v21, k22, v22, k23, v23, k24, v24, k25, v25, k26, v26, k27, v27, k28, v28, k29, v29, k30, v30)
		for k in pairs(t) do
			t[k] = nil
		end
		if k1 then t[k1] = v1
		if k2 then t[k2] = v2
		if k3 then t[k3] = v3
		if k4 then t[k4] = v4
		if k5 then t[k5] = v5
		if k6 then t[k6] = v6
		if k7 then t[k7] = v7
		if k8 then t[k8] = v8
		if k9 then t[k9] = v9
		if k10 then t[k10] = v10
		if k11 then t[k11] = v11
		if k12 then t[k12] = v12
		if k13 then t[k13] = v13
		if k14 then t[k14] = v14
		if k15 then t[k15] = v15
		if k16 then t[k16] = v16
		if k17 then t[k17] = v17
		if k18 then t[k18] = v18
		if k19 then t[k19] = v19
		if k20 then t[k20] = v20
		if k21 then t[k21] = v21
		if k22 then t[k22] = v22
		if k23 then t[k23] = v23
		if k24 then t[k24] = v24
		if k25 then t[k25] = v25
		if k26 then t[k26] = v26
		if k27 then t[k27] = v27
		if k28 then t[k28] = v28
		if k29 then t[k29] = v29
		if k30 then t[k30] = v30
		end end end end end end end end end end end end end end end end end end end end end end end end end end end end end end
		return t
	end
	return tmp[key]
end})

local headerSize, normalSize
if GameTooltipHeaderText then
	_,headerSize = GameTooltipHeaderText:GetFont()
else
	headerSize = 14
end
if GameTooltipText then
	_,normalSize = GameTooltipText:GetFont()
else
	normalSize = 12
end
local tooltip
local testString
local TabletData = {}
local Category = {}
local Line = {}
do
	local TabletData_mt = { __index = TabletData }
	function TabletData:new(tablet)
		if not testString then
			testString = UIParent:CreateFontString()
			testString:Hide()
		end
		local self = new()
		self.categories = new()
		self.id = 0
		self.width = 0--(MIN_TOOLTIP_SIZE - 20)*tablet.fontSizePercent
		self.tablet = tablet
		self.title = "Title"
		self.titleR, self.titleG, self.titleB = nil, nil, nil
		setmetatable(self, TabletData_mt)
		return self
	end

	function TabletData:del()
		for k, v in ipairs(self.categories) do
			v:del()
		end
		del(self.categories)
		del(self)
	end

	function TabletData:Display()
		if self.tablet == tooltip or self.tablet.registration.showTitleWhenDetached then
			local info = new(
				'hideBlankLine', true,
				'text', self.title,
				'justify', "CENTER",
				'font', GameTooltipHeaderText,
				'isTitle', true
			)
			if self.titleR then
				info.textR = self.titleR
				info.textG = self.titleG
				info.textB = self.titleB
			end
			self:AddCategory(info, 1)
			del(info)
		end
		if self.tablet == tooltip or self.tablet.registration.showHintWhenDetached then
			if self.hint then
				self:AddCategory(nil):AddLine(
					'text', HINT .. ": " .. self.hint,
					'textR', 0,
					'textG', 1,
					'textB', 0,
					'wrap', true
				)
			end
		end

		local tabletData = self.tabletData
		local width
		for k, v in ipairs(self.categories) do
			if v.columns <= 2 then
				width = v.x1
			else
				width = v.x1 + v.x2 + v.x3 + v.x4 + v.x5 + v.x6 + (v.columns - 1) * 20
			end
			if self.width < width then
				self.width = width
			end
		end

		local good = false
		local lastTitle = true
		for k, v in ipairs(self.categories) do
			if lastTitle then
				v.hideBlankLine = true
				lastTitle = false
			end
			if v:Display(self.tablet) then
				good = true
			end
			if v.isTitle then
				lastTitle = true
			end
		end
		if not good then
			if self.tablet == tooltip or not self.tablet.registration.hideWhenEmpty then
				local width
				local info = new(
					'hideBlankLine', true,
					'text', self.title,
					'justify', "CENTER",
					'font', GameTooltipHeaderText,
					'isTitle', true
				)
				local cat = self:AddCategory(info)
				del(info)
				self.width = self.categories[table.getn(self.categories)].x1
				cat:Display(self.tablet)
			else
				self.tablet:__Hide()
				self.tablet.tmpHidden = true
			end
		else
			self.tablet:__Show()
			self.tablet.tmpHidden = nil
		end
	end

	function TabletData:AddCategory(info, index)
		local made = false
		if not info then
			made = true
			info = new()
		end
		local cat = Category:new(self, info)
		if index then
			table.insert(self.categories, index, cat)
		else
			table.insert(self.categories, cat)
		end
		if made then
			del(info)
		end
		return cat
	end
	
	function TabletData:SetHint(hint)
		self.hint = hint
	end
	
	function TabletData:SetTitle(title)
		self.title = title or "Title"
	end
	
	function TabletData:SetTitleColor(r, g, b)
		self.titleR = r
		self.titleG = g
		self.titleB = b
	end
end
do
	local Category_mt = { __index = Category }
	function Category:new(tabletData, info, superCategory)
		local self = copy(info)
		if superCategory and not self.noInherit then
			self.superCategory = superCategory.superCategory
			for k, v in pairs(superCategory) do
				if string.find(k, "^child_") then
					local k = strsub(k, 7)
					if self[k] == nil then
						self[k] = v
					end
				end
			end
			self.columns = superCategory.columns
		else
			self.superCategory = self
		end
		self.tabletData = tabletData
		self.lines = new()
		if not self.columns then
			self.columns = 1
		end
		self.x1 = 0
		self.x2 = 0
		self.x3 = 0
		self.x4 = 0
		self.x5 = 0
		self.x6 = 0
		setmetatable(self, Category_mt)
		self.lastWasTitle = nil
		if self.text or self.text2 or self.text3 or self.text4 or self.text5 or self.text6 then
			local x = new(
				'category', category,
				'text', self.text,
				'textR', self.textR or 1,
				'textG', self.textG or 1,
				'textB', self.textB or 1,
				'fakeChild', true,
				'func', self.func,
				'arg1', info.arg1,
				'arg2', self.arg2,
				'arg3', self.arg3,
				'hasCheck', self.hasCheck,
				'checked', self.checked,
				'checkIcon', self.checkIcon,
				'isRadio', self.isRadio,
				'font', self.font,
				'size', self.size,
				'wrap', self.wrap,
				'catStart', true,
				'indentation', self.indentation,
				'noInherit', true,
				'justify', self.justify,
				'justify2', self.justify2,
				'justify3', self.justify3,
				'justify4', self.justify4,
				'justify5', self.justify5,
				'justify6', self.justify6
			)
			if self.isTitle then
				x.textR = self.textR or 1
				x.textG = self.textG or 0.823529
				x.textB = self.textB or 0
			else
				x.textR = self.textR or 1
				x.textG = self.textG or 1
				x.textB = self.textB or 1
			end
			x.text2 = self.text2
			x.text3 = self.text3
			x.text4 = self.text4
			x.text5 = self.text5
			x.text6 = self.text6
			x.text2R = self.text2R or self.textR2 or 1
			x.text2G = self.text2G or self.textG2 or 1
			x.text2B = self.text2B or self.textB2 or 1
			x.text3R = self.text3R or self.textR3 or 1
			x.text3G = self.text3G or self.textG3 or 1
			x.text3B = self.text3B or self.textB3 or 1
			x.text4R = self.text4R or self.textR4 or 1
			x.text4G = self.text4G or self.textG4 or 1
			x.text4B = self.text4B or self.textB4 or 1
			x.text5R = self.text5R or self.textR5 or 1
			x.text5G = self.text5G or self.textG5 or 1
			x.text5B = self.text5B or self.textB5 or 1
			x.text6R = self.text6R or self.textR6 or 1
			x.text6G = self.text6G or self.textG6 or 1
			x.text6B = self.text6B or self.textB6 or 1
			x.font2 = self.font2
			x.font3 = self.font3
			x.font4 = self.font4
			x.font5 = self.font5
			x.font6 = self.font6
			x.size2 = self.size2
			x.size3 = self.size3
			x.size4 = self.size4
			x.size5 = self.size5
			x.size6 = self.size6
			self:AddLine(x)
			del(x)
			self.lastWasTitle = true
		end
		return self
	end

	function Category:del()
		local prev = garbageLine
		for k, v in pairs(self.lines) do
			v:del()
		end
		del(self.lines)
		del(self)
	end

	function Category:AddLine(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20, k21, v21, k22, v22, k23, v23, k24, v24, k25, v25, k26, v26, k27, v27, k28, v28, k29, v29, k30, v30)
		self.lastWasTitle = nil
		local line
		if type(k1) == "table" then
			Line:new(self, k1, v1)
		else
			local info = new(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20, k21, v21, k22, v22, k23, v23, k24, v24, k25, v25, k26, v26, k27, v27, k28, v28, k29, v29, k30, v30)
			Line:new(self, info)
			del(info)
		end
	end

	function Category:AddCategory(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20, k21, v21, k22, v22, k23, v23, k24, v24, k25, v25, k26, v26, k27, v27, k28, v28, k29, v29, k30, v30)
		local lastWasTitle = self.lastWasTitle
		self.lastWasTitle = nil
		local info
		if type(k1) == "table" then
			info = k1
		else
			info = new(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20, k21, v21, k22, v22, k23, v23, k24, v24, k25, v25, k26, v26, k27, v27, k28, v28, k29, v29, k30, v30)
		end
		if lastWasTitle or table.getn(self.lines) == 0 then
			info.hideBlankLine = true
		end
		local cat = Category:new(self.tabletData, info, self)
		table.insert(self.lines, cat)
		return cat
	end

	function Category:HasChildren()
		local hasChildren = false
		for k, v in ipairs(self.lines) do
			if v.HasChildren then
				if v:HasChildren() then
					return true
				end
			end
			if not v.fakeChild then
				return true
			end
		end
		return false
	end

	local lastWasTitle = false
	function Category:Display(tablet)
		if not self.isTitle and not self.showWithoutChildren and not self:HasChildren() then
			return false
		end
		if not self.hideBlankLine and not lastWasTitle then
			local info = new(
				'blank', true,
				'fakeChild', true
			)
			self:AddLine(info, 1)
			del(info)
		end
		local good = false
		if table.getn(self.lines) > 0 then
			self.tabletData.id = self.tabletData.id + 1
			self.id = self.tabletData.id
			for k, v in ipairs(self.lines) do
				if v:Display(tablet) then
					good = true
				end
			end
		end
		lastWasTitle = self.isTitle
		return good
	end
end
do
	local Line_mt = { __index = Line }
	function Line:new(category, info, position)
		local self = copy(info)
		if not info.noInherit then
			for k, v in pairs(category) do
				if string.find(k, "^child_") then
					local k = strsub(k, 7)
					if self[k] == nil then
						self[k] = v
					end
				end
			end
		end
		self.category = category
		if position then
			table.insert(category.lines, position, self)
		else
			table.insert(category.lines, self)
		end
		setmetatable(self, Line_mt)
		local columns = category.columns
		if columns == 1 then
			if not self.justify then
				self.justify = "LEFT"
			end
		elseif columns == 2 then
			self.justify = "LEFT"
			self.justify2 = "RIGHT"
			if self.wrap then
				self.wrap2 = false
			end
		elseif columns == 3 then
			if not self.justify then
				self.justify = "LEFT"
			end
			if not self.justify2 then
				self.justify2 = "CENTER"
			end
			if not self.justify3 then
				self.justify3 = "RIGHT"
			end
			if self.wrap then
				self.wrap2 = false
				self.wrap3 = false
			elseif self.wrap2 then
				self.wrap3 = false
			end
		elseif columns == 4 then
			if not self.justify then
				self.justify = "LEFT"
			end
			if not self.justify2 then
				self.justify2 = "CENTER"
			end
			if not self.justify3 then
				self.justify3 = "CENTER"
			end
			if not self.justify4 then
				self.justify4 = "RIGHT"
			end
			if self.wrap then
				self.wrap2 = false
				self.wrap3 = false
				self.wrap4 = false
			elseif self.wrap2 then
				self.wrap3 = false
				self.wrap4 = false
			elseif self.wrap3 then
				self.wrap4 = false
			end
		elseif columns == 5 then
			if not self.justify then
				self.justify = "LEFT"
			end
			if not self.justify2 then
				self.justify2 = "CENTER"
			end
			if not self.justify3 then
				self.justify3 = "CENTER"
			end
			if not self.justify4 then
				self.justify4 = "CENTER"
			end
			if not self.justify5 then
				self.justify5 = "RIGHT"
			end
			if self.wrap then
				self.wrap2 = false
				self.wrap3 = false
				self.wrap4 = false
				self.wrap5 = false
			elseif self.wrap2 then
				self.wrap3 = false
				self.wrap4 = false
				self.wrap5 = false
			elseif self.wrap3 then
				self.wrap4 = false
				self.wrap5 = false
			elseif self.wrap4 then
				self.wrap5 = false
			end
		elseif columns == 6 then
			if not self.justify then
				self.justify = "LEFT"
			end
			if not self.justify2 then
				self.justify2 = "CENTER"
			end
			if not self.justify3 then
				self.justify3 = "CENTER"
			end
			if not self.justify4 then
				self.justify4 = "CENTER"
			end
			if not self.justify5 then
				self.justify5 = "CENTER"
			end
			if not self.justify6 then
				self.justify6 = "RIGHT"
			end
			if self.wrap then
				self.wrap2 = false
				self.wrap3 = false
				self.wrap4 = false
				self.wrap5 = false
				self.wrap6 = false
			elseif self.wrap2 then
				self.wrap3 = false
				self.wrap4 = false
				self.wrap5 = false
				self.wrap6 = false
			elseif self.wrap3 then
				self.wrap4 = false
				self.wrap5 = false
				self.wrap6 = false
			elseif self.wrap4 then
				self.wrap5 = false
				self.wrap6 = false
			elseif self.wrap5 then
				self.wrap6 = false
			end
		end
		if self.textR2 then
			self.text2R, self.textR2 = self.text2R or self.textR2
			self.text2G, self.textG2 = self.text2G or self.textG2
			self.text2B, self.textB2 = self.text2B or self.textB2
			if self.textR3 then
				self.text3R, self.textR3 = self.text3R or self.textR3
				self.text3G, self.textG3 = self.text3G or self.textG3
				self.text3B, self.textB3 = self.text3B or self.textB3
				if self.textR4 then
					self.text4R, self.textR4 = self.text4R or self.textR4
					self.text4G, self.textG4 = self.text4G or self.textG4
					self.text4B, self.textB4 = self.text4B or self.textB4
					if self.textR5 then
						self.text5R, self.textR5 = self.text5R or self.textR5
						self.text5G, self.textG5 = self.text5G or self.textG5
						self.text5B, self.textB5 = self.text5B or self.textB5
						if self.textR5 then
							self.text6R, self.textR6 = self.text6R or self.textR6
							self.text6G, self.textG6 = self.text6G or self.textG6
							self.text6B, self.textB6 = self.text6B or self.textB6
						end
					end
				end
			end
		end
		if not self.indentation or self.indentation < 0 then
			self.indentation = 0
		end
		if not self.font then
			self.font = GameTooltipText
		end
		if not self.font2 then
			self.font2 = self.font
		end
		if not self.font3 then
			self.font3 = self.font
		end
		if not self.font4 then
			self.font4 = self.font
		end
		if not self.font5 then
			self.font5 = self.font
		end
		if not self.font6 then
			self.font6 = self.font
		end
		if not self.size then
			_,self.size = self.font:GetFont()
		end
		if not self.size2 then
			_,self.size2 = self.font2:GetFont()
		end
		if not self.size3 then
			_,self.size3 = self.font3:GetFont()
		end
		if not self.size4 then
			_,self.size4 = self.font4:GetFont()
		end
		if not self.size5 then
			_,self.size5 = self.font5:GetFont()
		end
		if not self.size6 then
			_,self.size6 = self.font6:GetFont()
		end

		local fontSizePercent = category.tabletData.tablet.fontSizePercent
		local w = 0
		self.checkWidth = 0
		if self.text then
			if not self.wrap then
				testString:SetWidth(0)
				testString:SetFontObject(self.font)
				local font,_,flags = testString:GetFont()
				testString:SetFont(font, self.size * fontSizePercent, flags)
				testString:SetText(self.text)
				local checkWidth = self.hasCheck and self.size * fontSizePercent or 0
				self.checkWidth = checkWidth
				w = testString:GetWidth() + self.indentation * fontSizePercent + checkWidth + TESTSTRING_EXTRA_WIDTH
				if category.superCategory.x1 < w then
					category.superCategory.x1 = w
				end
			else
				if columns == 1 then
					testString:SetWidth(0)
					testString:SetFontObject(self.font)
					local font,_,flags = testString:GetFont()
					testString:SetFont(font, self.size * fontSizePercent, flags)
					testString:SetText(self.text)
					local checkWidth = self.hasCheck and self.size * fontSizePercent or 0
					self.checkWidth = checkWidth
					w = testString:GetWidth() + self.indentation * fontSizePercent + checkWidth + TESTSTRING_EXTRA_WIDTH
					if w > (MIN_TOOLTIP_SIZE - 20) * fontSizePercent then
						w = (MIN_TOOLTIP_SIZE - 20) * fontSizePercent
					end
				else
					w = MIN_TOOLTIP_SIZE * fontSizePercent / 2
				end
				if category.superCategory.x1 < w then
					category.superCategory.x1 = w
				end
			end
		end
		if columns == 2 and self.text2 then
			if not self.wrap2 then
				testString:SetWidth(0)
				testString:SetFontObject(self.font2)
				local font,_,flags = testString:GetFont()
				testString:SetFont(font, self.size2 * fontSizePercent, flags)
				testString:SetText(self.text2)
				w = w + 40 * fontSizePercent + testString:GetWidth() + TESTSTRING_EXTRA_WIDTH
				if category.superCategory.x1 < w then
					category.superCategory.x1 = w
				end
			else
				w = w + 40 * fontSizePercent + MIN_TOOLTIP_SIZE * fontSizePercent / 2
				if category.superCategory.x1 < w then
					category.superCategory.x1 = w
				end
			end
		elseif columns >= 3 then
			if self.text2 then
				if not self.wrap2 then
					testString:SetWidth(0)
					testString:SetFontObject(self.font2)
					local font,_,flags = testString:GetFont()
					testString:SetFont(font, self.size2 * fontSizePercent, flags)
					testString:SetText(self.text2)
					local w = testString:GetWidth() + TESTSTRING_EXTRA_WIDTH
					if category.superCategory.x2 < w then
						category.superCategory.x2 = w
					end
				else
					local w = MIN_TOOLTIP_SIZE / 2
					if category.superCategory.x2 < w then
						category.superCategory.x2 = w
					end
				end
			end
			if self.text3 then
				if not self.wrap3 then
					testString:SetWidth(0)
					testString:SetFontObject(self.font3)
					local font,_,flags = testString:GetFont()
					testString:SetFont(font, self.size3 * fontSizePercent, flags)
					testString:SetText(self.text3)
					local w = testString:GetWidth() + TESTSTRING_EXTRA_WIDTH
					if category.superCategory.x3 < w then
						category.superCategory.x3 = w
					end
				else
					local w = MIN_TOOLTIP_SIZE / 2
					if category.superCategory.x3 < w then
						category.superCategory.x3 = w
					end
				end
			end
			if columns >= 4 then
				if self.text4 then
					if not self.wrap4 then
						testString:SetWidth(0)
						testString:SetFontObject(self.font4)
						local font,_,flags = testString:GetFont()
						testString:SetFont(font, self.size4 * fontSizePercent, flags)
						testString:SetText(self.text4)
						w = testString:GetWidth() + TESTSTRING_EXTRA_WIDTH
						if category.superCategory.x4 < w then
							category.superCategory.x4 = w
						end
					else
						local w = MIN_TOOLTIP_SIZE / 2
						if category.superCategory.x4 < w then
							category.superCategory.x4 = w
						end
					end
				end
				if columns >= 5 then
					if self.text5 then
						if not self.wrap5 then
							testString:SetWidth(0)
							testString:SetFontObject(self.font5)
							local font,_,flags = testString:GetFont()
							testString:SetFont(font, self.size5 * fontSizePercent, flags)
							testString:SetText(self.text5)
							w = testString:GetWidth() + TESTSTRING_EXTRA_WIDTH
							if category.superCategory.x5 < w then
								category.superCategory.x5 = w
							end
						else
							local w = MIN_TOOLTIP_SIZE / 2
							if category.superCategory.x5 < w then
								category.superCategory.x5 = w
							end
						end
					end
					if columns >= 6 then
						if self.text6 then
							if not self.wrap6 then
								testString:SetWidth(0)
								testString:SetFontObject(self.font6)
								local font,_,flags = testString:GetFont()
								testString:SetFont(font, self.size6 * fontSizePercent, flags)
								testString:SetText(self.text6)
								w = testString:GetWidth() + TESTSTRING_EXTRA_WIDTH
								if category.superCategory.x6 < w then
									category.superCategory.x6 = w
								end
							else
								local w = MIN_TOOLTIP_SIZE / 2
								if category.superCategory.x6 < w then
									category.superCategory.x6 = w
								end
							end
						end
					end
				end
			end
		end
		return self
	end

	function Line:del()
		del(self)
	end

	function Line:Display(tablet)
		tablet:AddLine(self)
		return true
	end
end

local function button_OnEnter()
	if type(this.self:GetScript("OnEnter")) == "function" then
		this.self:GetScript("OnEnter")()
	end
	this.highlight:Show()
end

local function button_OnLeave()
	if type(this.self:GetScript("OnLeave")) == "function" then
		this.self:GetScript("OnLeave")()
	end
	this.highlight:Hide()
end

local function NewLine(self)
	if self.maxLines <= self.numLines then
		self.maxLines = self.maxLines + 1
		local button = CreateFrame("Button", nil, self)
		button.indentation = 0
		local check = button:CreateTexture(nil, "ARTWORK")
		local left = button:CreateFontString(nil, "ARTWORK")
		local right = button:CreateFontString(nil, "ARTWORK")
		local third = button:CreateFontString(nil, "ARTWORK")
		local fourth = button:CreateFontString(nil, "ARTWORK")
		local fifth = button:CreateFontString(nil, "ARTWORK")
		local sixth = button:CreateFontString(nil, "ARTWORK")
		local highlight = button:CreateTexture(nil, "BACKGROUND")
		highlight:SetTexture("Interface\\QuestFrame\\UI-QuestTitleHighlight")
		button.highlight = highlight
		highlight:SetBlendMode("ADD")
		highlight:SetAllPoints(button)
		highlight:Hide()
		table.insert(self.buttons, button)
		table.insert(self.checks, check)
		table.insert(self.lefts, left)
		table.insert(self.rights, right)
		table.insert(self.thirds, third)
		table.insert(self.fourths, fourth)
		table.insert(self.fifths, fifth)
		table.insert(self.sixths, sixth)
		left:SetWidth(0)
		if self.maxLines == 1 then
			left:SetFontObject(GameTooltipHeaderText)
			right:SetFontObject(GameTooltipHeaderText)
			third:SetFontObject(GameTooltipHeaderText)
			fourth:SetFontObject(GameTooltipHeaderText)
			fifth:SetFontObject(GameTooltipHeaderText)
			sixth:SetFontObject(GameTooltipHeaderText)
			left:SetJustifyH("CENTER")
			button:SetPoint("TOPLEFT", self, "TOPLEFT", 8, -10)
		else
			left:SetFontObject(GameTooltipText)
			right:SetFontObject(GameTooltipText)
			third:SetFontObject(GameTooltipText)
			fourth:SetFontObject(GameTooltipText)
			fifth:SetFontObject(GameTooltipText)
			sixth:SetFontObject(GameTooltipText)
			button:SetPoint("TOPLEFT", self.buttons[self.maxLines - 1], "BOTTOMLEFT", 0, -2)
		end
		button:SetScript("OnEnter", button_OnEnter)
		button:SetScript("OnLeave", button_OnLeave)
		button.check = check
		button.self = self
		button:SetPoint("RIGHT", self, "RIGHT", -12, 0)
		check.shown = false
		check:SetPoint("TOPLEFT", button, "TOPLEFT")
		left:SetPoint("TOPLEFT", check, "TOPLEFT")
		right:SetPoint("TOPLEFT", left, "TOPRIGHT", 40 * self.fontSizePercent, 0)
		third:SetPoint("TOPLEFT", right, "TOPRIGHT", 20 * self.fontSizePercent, 0)
		fourth:SetPoint("TOPLEFT", third, "TOPRIGHT", 20 * self.fontSizePercent, 0)
		fifth:SetPoint("TOPLEFT", fourth, "TOPRIGHT", 20 * self.fontSizePercent, 0)
		sixth:SetPoint("TOPLEFT", fifth, "TOPRIGHT", 20 * self.fontSizePercent, 0)
		right:SetJustifyH("RIGHT")
		local _,size = GameTooltipText:GetFont()
		check:SetHeight(size * 1.5)
		check:SetWidth(size * 1.5)
		check:SetTexture("Interface\\Buttons\\UI-CheckBox-Check")
		check:SetAlpha(0)
		if not button.clicked then
			button:SetScript("OnMouseWheel", self:GetScript("OnMouseWheel"))
			button:EnableMouseWheel(true)
			button:Hide()
		end
		check:Show()
		left:Hide()
		right:Hide()
		third:Hide()
		fourth:Hide()
		fifth:Hide()
		sixth:Hide()
	end
end
NewLine = wrap(NewLine, "NewLine")

local function GetMaxLinesPerScreen(self)
	if self == tooltip then
		return floor(50 / self.fontSizePercent)
	else
		return floor(30 / self.fontSizePercent)
	end
end
GetMaxLinesPerScreen = wrap(GetMaxLinesPerScreen, "GetMaxLinesPerScreen")

local detachedTooltips = {}
local AcquireDetachedFrame, ReleaseDetachedFrame
local function AcquireFrame(self, registration, data, detachedData)
	if not detachedData then
		detachedData = data
	end
	if tooltip then
		tooltip.data = data
		tooltip.detachedData = detachedData
		local fontSizePercent = tooltip.data and tooltip.data.fontSizePercent or 1
		local transparency = tooltip.data and tooltip.data.transparency or 0.75
		local r = tooltip.data and tooltip.data.r or 0
		local g = tooltip.data and tooltip.data.g or 0
		local b = tooltip.data and tooltip.data.b or 0
		tooltip:SetFontSizePercent(fontSizePercent)
		tooltip:SetTransparency(transparency)
		tooltip:SetColor(r, g, b)
	else
		tooltip = CreateFrame("Frame", "Tablet20Frame", UIParent)
		self.tooltip = tooltip
		tooltip.data = data
		tooltip.detachedData = detachedData
		tooltip:EnableMouse(true)
		tooltip:EnableMouseWheel(true)
		tooltip:SetFrameStrata("TOOLTIP")
		tooltip:SetFrameLevel(10)
		local backdrop = new(
			'bgFile', "Interface\\Buttons\\WHITE8X8",
			'edgeFile', "Interface\\Tooltips\\UI-Tooltip-Border",
			'tile', true,
			'tileSize', 16,
			'edgeSize', 16,
			'insets', new(
				'left', 5,
				'right', 5,
				'top', 5,
				'bottom', 5
			)
		)
		tooltip:SetBackdrop(backdrop)
		del(backdrop.insets)
		del(backdrop)
		tooltip:SetBackdropColor(0, 0, 0, 1)

		tooltip.numLines = 0
		tooltip.owner = nil
		tooltip.fontSizePercent = tooltip.data and tooltip.data.fontSizePercent or 1
		tooltip.maxLines = 0
		tooltip.buttons = {}
		tooltip.checks = {}
		tooltip.lefts = {}
		tooltip.rights = {}
		tooltip.thirds = {}
		tooltip.fourths = {}
		tooltip.fifths = {}
		tooltip.sixths = {}
		tooltip.transparency = tooltip.data and tooltip.data.transparency or 0.75
		tooltip:SetBackdropColor(0, 0, 0, tooltip.transparency)
		tooltip:SetBackdropBorderColor(1, 1, 1, tooltip.transparency)
		tooltip.scroll = 0

		tooltip:SetScript("OnUpdate", function()
			if not tooltip.updating and not tooltip.enteredFrame then
				tooltip.scroll = 0
				tooltip:Hide()
				tooltip.registration.tooltip = nil
				tooltip.registration = nil
			end
		end)

		tooltip:SetScript("OnEnter", function()
			if tooltip.clickable then
				tooltip.enteredFrame = true
			end
		end)

		tooltip:SetScript("OnLeave", function()
			if not tooltip.updating then
				tooltip.enteredFrame = false
			end
		end)

		tooltip:SetScript("OnMouseWheel", function()
			tooltip.updating = true
			tooltip:Scroll(arg1 < 0)
			tooltip.updating = false
		end)

		NewLine(tooltip)

		tooltip.scrollUp = tooltip:CreateFontString(nil, "ARTWORK")
		tooltip.scrollUp:SetPoint("TOPLEFT", tooltip.buttons[1], "BOTTOMLEFT", 0, -2)
		tooltip.scrollUp:SetPoint("RIGHT", tooltip, "RIGHT", 0, -10)
		tooltip.scrollUp:SetFontObject(GameTooltipText)
		tooltip.scrollUp:Hide()
		local font,_,flags = tooltip.scrollUp:GetFont()
		tooltip.scrollUp:SetFont(font, normalSize * tooltip.fontSizePercent, flags)
		tooltip.scrollUp:SetJustifyH("CENTER")
		tooltip.scrollUp:SetTextColor(1, 0.823529, 0)
		tooltip.scrollUp:SetText(" ")

		tooltip.scrollDown = tooltip:CreateFontString(nil, "ARTWORK")
		tooltip.scrollDown:SetPoint("TOPLEFT", tooltip.buttons[1], "BOTTOMLEFT", 0, -2)
		tooltip.scrollDown:SetPoint("RIGHT", tooltip, "RIGHT", 0, -10)
		tooltip.scrollDown:SetFontObject(GameTooltipText)
		tooltip.scrollDown:Hide()
		local font,_,flags = tooltip.scrollUp:GetFont()
		tooltip.scrollDown:SetFont(font, normalSize * tooltip.fontSizePercent, flags)
		tooltip.scrollDown:SetJustifyH("CENTER")
		tooltip.scrollDown:SetTextColor(1, 0.823529, 0)
		tooltip.scrollDown:SetText(" ")

		function tooltip:SetOwner(o)
			self:Hide(o)
			self.owner = o
		end
		tooltip.SetOwner = wrap(tooltip.SetOwner, "tooltip:SetOwner")

		function tooltip:IsOwned(o)
			return self.owner == o
		end
		tooltip.IsOwned = wrap(tooltip.IsOwned, "tooltip:IsOwned")

		function tooltip:ClearLines(hide)
			CleanCategoryPool(self)
			for i = 1, self.numLines do
				local button = self.buttons[i]
				local check = self.checks[i]
				if not button.clicked or hide then
					button:Hide()
				end
				check.shown = false
				check:SetAlpha(0)
			end
			self.numLines = 0
		end
		tooltip.ClearLines = wrap(tooltip.ClearLines, "tooltip:ClearLines")

		function tooltip:NumLines()
			return self.numLines
		end

		local lastWidth
		local old_tooltip_Hide = tooltip.Hide
		tooltip.__Hide = old_tooltip_Hide
		function tooltip:Hide(newOwner)
			if self == tooltip or newOwner == nil then
				old_tooltip_Hide(self)
			end
			self:ClearLines(true)
			self.owner = nil
			self.lastWidth = nil
			self.tmpHidden = nil
		end
		tooltip.Hide = wrap(tooltip.Hide, "tooltip:Hide")

		local old_tooltip_Show = tooltip.Show
		tooltip.__Show = old_tooltip_Show
		function tooltip:Show(tabletData)
			if self.owner == nil or self.notInUse then
				return
			end
			if not self.tmpHidden then
				old_tooltip_Show(self)
			end

			local maxWidth = tabletData and tabletData.width or self:GetWidth() - 20
			local hasWrap = false
			local screenWidth = GetScreenWidth()
			local scrollMax = self.numLines
			if scrollMax > GetMaxLinesPerScreen(self) + self.scroll then
				scrollMax = GetMaxLinesPerScreen(self) + self.scroll
			end
			local numColumns

			local height = 20
			if scrollMax ~= self.numLines then
				self.scrollDown:SetWidth(maxWidth)
				height = height + self.scrollDown:GetHeight() + 2
			end
			if self.scroll ~= 0 then
				self.scrollUp:SetWidth(maxWidth)
				height = height + self.scrollUp:GetHeight() + 2
			end
			self:SetWidth(maxWidth + 20)

			local tmp = self.scroll + 1
			if tmp ~= 1 then
				tmp = tmp + 1
			end
			for i = 1, self.numLines do
				if i < tmp or i > scrollMax or (i == scrollMax and i ~= self.numLines) then
					self.buttons[i]:ClearAllPoints()
					self.buttons[i]:Hide()
				else
					local button = self.buttons[i]
					local left = self.lefts[i]
					local right = self.rights[i]
					local check = self.checks[i]
					button:SetWidth(maxWidth)
					button:SetHeight(math.max(left:GetHeight(), right:GetHeight()))
					height = height + button:GetHeight() + 2
					if i == self.scroll + 1 then
						button:SetPoint("TOPLEFT", self, "TOPLEFT", 10, -10)
					else
						button:SetPoint("TOPLEFT", self.buttons[i - 1], "BOTTOMLEFT", 0, -2)
					end
					if button.clicked then
						check:SetPoint("TOPLEFT", button, "TOPLEFT", button.indentation * self.fontSizePercent + (check.width - check:GetWidth()) / 2 + 1, -1)
					else
						check:SetPoint("TOPLEFT", button, "TOPLEFT", button.indentation * self.fontSizePercent + (check.width - check:GetWidth()) / 2, 0)
					end
					button:Show()
				end
			end
			if self.scroll ~= 0 then
				self.scrollUp:SetPoint("TOPLEFT", self, "TOPLEFT", 10, -10)
				self.buttons[self.scroll + 2]:SetPoint("TOPLEFT", self.scrollUp, "BOTTOMLEFT", 0, -2)
				self.scrollUp:SetText(SCROLL_UP .. " (" .. self.scroll + 2 .. " / " .. self.numLines .. ")")
				self.scrollUp:Show()
			else
				self.scrollUp:Hide()
			end
			if scrollMax ~= self.numLines and self.buttons[scrollMax - 1] then
				self.scrollDown:SetPoint("TOPLEFT", self.buttons[scrollMax - 1], "BOTTOMLEFT", 0, -2)
				self.scrollDown:SetText(SCROLL_DOWN .. " (" .. scrollMax - 1 .. " / " .. self.numLines .. ")")
				self.scrollDown:Show()
			else
				self.scrollDown:Hide()
			end
			self:SetHeight(height)
		end
		tooltip.Show = wrap(tooltip.Show, "tooltip:Show")

		local lastMouseDown
		local function button_OnClick()
			if this.self:HasScript("OnClick") and type(this.self:GetScript("OnClick")) == "function" then
				this.self:GetScript("OnClick")()
			end
			if arg1 == "RightButton" then
				if this.self:HasScript("OnClick") and type(this.self:GetScript("OnClick")) == "function" then
					this.self:GetScript("OnClick")()
				end
			elseif arg1 == "LeftButton" then
				if this.self.preventClick == nil or GetTime() > this.self.preventClick and GetTime() < lastMouseDown + 0.5 then
					this.self.preventClick = nil
					this.self.updating = true
					this.self.preventRefresh = true
					this.func(this.a1, this.a2, this.a3)
					if this.self then
						this.self.preventRefresh = false
						this.self:children()
						this.self.updating = false
					end
				end
			end
		end
		local function button_OnMouseUp()
			if this.self:HasScript("OnMouseUp") and type(this.self:GetScript("OnMouseUp")) == "function" then
				this.self:GetScript("OnMouseUp")()
			end
			if arg1 ~= "RightButton" then
				if this.clicked then
					local a,b,c,d,e = this.check:GetPoint(1)
					this.check:SetPoint(a,b,c,d-1,e+1)
					this.clicked = false
				end
			end
		end
		local function button_OnMouseDown()
			if this.self:HasScript("OnMouseDown") and type(this.self:GetScript("OnMouseDown")) == "function" then
				this.self:GetScript("OnMouseDown")()
			end
			lastMouseDown = GetTime()
			if arg1 ~= "RightButton" then
				local a,b,c,d,e = this.check:GetPoint(1)
				this.check:SetPoint(a,b,c,d+1,e-1)
				this.clicked = true
			end
		end
		function tooltip:AddLine(info)
			local category = info.category.superCategory
			local maxWidth = category.tabletData.width
			local text = info.blank and "\n" or info.text
			local id = info.id
			local func = info.func
			local checked = info.checked
			local isRadio = info.isRadio
			local checkTexture = info.checkTexture
			local fontSizePercent = self.fontSizePercent
			if not info.font then
				info.font = GameTooltipText
			end
			if not info.size then
				_,info.size = info.font:GetFont()
			end
			local catStart = false
			local columns = category and category.columns or 1
			local x1, x2, x3, x4, x5, x6
			if category then
				x1, x2, x3, x4, x5, x6 = category.x1, category.x2, category.x3, category.x4, category.x5, category.x6
			else
				x1, x2, x3, x4, x5, x6 = 0, 0, 0, 0, 0, 0
			end
			if info.isTitle then
				justAddedTitle = true
			end

			self.numLines = self.numLines + 1
			NewLine(self)
			self.lefts[self.numLines]:Show()
			self.buttons[self.numLines]:Show()
			num = self.numLines

			local button = self.buttons[num]
			button.indentation = info.indentation
			local left = self.lefts[num]
			local right = self.rights[num]
			local third = self.thirds[num]
			local fourth = self.fourths[num]
			local fifth = self.fifths[num]
			local sixth = self.sixths[num]
			local check = self.checks[num]
			do -- if columns >= 1 then
				left:SetFontObject(info.font)
				left:SetText(text)
				left:Show()
				if info.textR and info.textG and info.textB then
					left:SetTextColor(info.textR, info.textG, info.textB)
				else
					left:SetTextColor(1, 0.823529, 0)
				end
				local a,_,b = left:GetFont()
				left:SetFont(a, info.size * fontSizePercent, b)
				left:SetJustifyH(info.justify)
				if columns < 2 then
					right:SetText(nil)
					right:Hide()
					right:SetPoint("TOPLEFT", left, "TOPRIGHT", 40 * fontSizePercent, 0)
					right:SetPoint("TOPRIGHT", button, "TOPRIGHT", -5, 0)
					third:SetText(nil)
					third:Hide()
					fourth:SetText(nil)
					fourth:Hide()
					fifth:SetText(nil)
					fifth:Hide()
					sixth:SetText(nil)
					sixth:Hide()
				else
					right:SetFontObject(info.font2)
					right:SetText(info.text2)
					right:Show()
					if info.text2R and info.text2G and info.text2B then
						right:SetTextColor(info.text2R, info.text2G, info.text2B)
					else
						right:SetTextColor(1, 0.823529, 0)
					end
					local a,_,b = right:GetFont()
					right:SetFont(a, info.size2 * fontSizePercent, b)
					right:SetJustifyH(info.justify2)
					if columns < 3 then
						right:SetPoint("TOPLEFT", left, "TOPRIGHT", 40 * fontSizePercent, 0)
						right:SetPoint("TOPRIGHT", button, "TOPRIGHT", -5, 0)
						third:SetText(nil)
						third:Hide()
						fourth:SetText(nil)
						fourth:Hide()
						fifth:SetText(nil)
						fifth:Hide()
						sixth:SetText(nil)
						sixth:Hide()
					else
						third:SetFontObject(info.font3)
						third:SetText(info.text3)
						third:Show()
						if info.text3R and info.text3G and info.text3B then
							third:SetTextColor(info.text3R, info.text3G, info.text3B)
						else
							third:SetTextColor(1, 0.823529, 0)
						end
						local a,_,b = third:GetFont()
						third:SetFont(a, info.size3 * fontSizePercent, b)
						right:ClearAllPoints()
						right:SetPoint("TOPLEFT", left, "TOPRIGHT", 20 * fontSizePercent, 0)
						third:SetJustifyH(info.justify3)
						if columns < 4 then
							fourth:SetText(nil)
							fourth:Hide()
							fifth:SetText(nil)
							fifth:Hide()
							sixth:SetText(nil)
							sixth:Hide()
						else
							fourth:SetFontObject(info.font4)
							fourth:SetText(info.text4)
							fourth:Show()
							if info.text4R and info.text4G and info.text4B then
								fourth:SetTextColor(info.text4R, info.text4G, info.text4B)
							else
								fourth:SetTextColor(1, 0.823529, 0)
							end
							local a,_,b = fourth:GetFont()
							fourth:SetFont(a, info.size4 * fontSizePercent, b)
							fourth:SetJustifyH(info.justify4)
							if columns < 5 then
								fifth:SetText(nil)
								fifth:Hide()
								sixth:SetText(nil)
								sixth:Hide()
							else
								fifth:SetFontObject(info.font5)
								fifth:SetText(info.text5)
								fifth:Show()
								if info.text5R and info.text5G and info.text5B then
									fifth:SetTextColor(info.text5R, info.text5G, info.text5B)
								else
									fifth:SetTextColor(1, 0.823529, 0)
								end
								local a,_,b = fourth:GetFont()
								fifth:SetFont(a, info.size5 * fontSizePercent, b)
								fifth:SetJustifyH(info.justify5)
								if columns < 6 then
									sixth:SetText(nil)
									sixth:Hide()
								else
									sixth:SetFontObject(info.font6)
									sixth:SetText(info.text6)
									sixth:Show()
									if info.text5R and info.text6G and info.text6B then
										sixth:SetTextColor(info.text6R, info.text6G, info.text6B)
									else
										sixth:SetTextColor(1, 0.823529, 0)
									end
									local a,_,b = fourth:GetFont()
									sixth:SetFont(a, info.size6 * fontSizePercent, b)
									sixth:SetJustifyH(info.justify6)
								end
							end
						end
					end
				end
			end

			check:SetWidth(info.size)
			check:SetHeight(info.size)
			check.width = info.size
			if info.hasCheck then
				check.shown = true
				check:Show()
				if isRadio then
					check:SetTexture(info.checkIcon or "Interface\\Buttons\\UI-RadioButton")
					if info.checked then
						check:SetAlpha(1)
						check:SetTexCoord(0.25, 0.5, 0, 1)
					else
						check:SetAlpha(self.transparency)
						check:SetTexCoord(0, 0.25, 0, 1)
					end
				else
					if info.checkIcon then
						check:SetTexture(info.checkIcon)
						if string.sub(info.checkIcon, 1, 16) == "Interface\\Icons\\" then
							check:SetTexCoord(0.05, 0.95, 0.05, 0.95)
						else
							check:SetTexCoord(0, 1, 0, 1)
						end
					else
						check:SetTexture("Interface\\Buttons\\UI-CheckBox-Check")
						check:SetWidth(info.size * 1.5)
						check:SetHeight(info.size * 1.5)
						check.width = info.size * 1.2
						check:SetTexCoord(0, 1, 0, 1)
					end
					check:SetAlpha(info.checked and 1 or 0)
				end
				left:SetPoint("TOPLEFT", check, "TOPLEFT", check.width, 0)
			else
				left:SetPoint("TOPLEFT", check, "TOPLEFT")
			end
			if columns == 1 then
				left:SetWidth(maxWidth)
			elseif columns == 2 then
				left:SetWidth(0)
				right:SetWidth(0)
				if info.wrap then
					left:SetWidth(maxWidth - right:GetWidth() - 40 * fontSizePercent)
				elseif info.wrap2 then
					right:SetWidth(maxWidth - left:GetWidth() - 40 * fontSizePercent)
				end
				right:ClearAllPoints()
				right:SetPoint("TOPRIGHT", button, "TOPRIGHT", 0, 0)
				if not info.text2 then
					left:SetJustifyH(info.justify or "LEFT")
				end
			elseif columns == 3 then
				left:SetWidth(x1 - info.checkWidth)
				right:SetWidth(x2)
				third:SetWidth(x3)
				right:ClearAllPoints()
				local num = (category.tabletData.width - x1 - x2 - x3) / 2
				right:SetPoint("TOPLEFT", left, "TOPRIGHT", num, 0)
				third:SetPoint("TOPLEFT", right, "TOPRIGHT", num, 0)
			elseif columns == 4 then
				left:SetWidth(x1 - info.checkWidth)
				right:SetWidth(x2)
				third:SetWidth(x3)
				fourth:SetWidth(x4)
				local num = (category.tabletData.width - x1 - x2 - x3 - x4) / 3
				right:SetPoint("TOPLEFT", left, "TOPRIGHT", num, 0)
				third:SetPoint("TOPLEFT", right, "TOPRIGHT", num, 0)
				fourth:SetPoint("TOPLEFT", third, "TOPRIGHT", num, 0)
			elseif columns == 5 then
				left:SetWidth(x1 - info.checkWidth)
				right:SetWidth(x2)
				third:SetWidth(x3)
				fourth:SetWidth(x4)
				fifth:SetWidth(x5)
				local num = (category.tabletData.width - x1 - x2 - x3 - x4 - x5) / 4
				right:SetPoint("TOPLEFT", left, "TOPRIGHT", num, 0)
				third:SetPoint("TOPLEFT", right, "TOPRIGHT", num, 0)
				fourth:SetPoint("TOPLEFT", third, "TOPRIGHT", num, 0)
				fifth:SetPoint("TOPLEFT", fourth, "TOPRIGHT", num, 0)
			elseif columns == 6 then
				left:SetWidth(x1 - info.checkWidth)
				right:SetWidth(x2)
				third:SetWidth(x3)
				fourth:SetWidth(x4)
				fifth:SetWidth(x5)
				sixth:SetWidth(x6)
				local num = (category.tabletData.width - x1 - x2 - x3 - x4 - x5 - x6) / 5
				right:SetPoint("TOPLEFT", left, "TOPRIGHT", num, 0)
				third:SetPoint("TOPLEFT", right, "TOPRIGHT", num, 0)
				fourth:SetPoint("TOPLEFT", third, "TOPRIGHT", num, 0)
				fifth:SetPoint("TOPLEFT", fourth, "TOPRIGHT", num, 0)
				sixth:SetPoint("TOPLEFT", fifth, "TOPRIGHT", num, 0)
			end
			if not self.locked or IsAltKeyDown() then
				local func = info.func
				if func then
					if type(func) == "string" then
						Tablet:assert(type(info.arg1) == "table", "Cannot call method " .. info.func .. " on a non-table")
						func = info.arg1[func]
						Tablet:assert(type(func) == "function", "Method " .. info.func .. " nonexistant")
					end
					Tablet:assert(type(func) == "function", "func must be a function or method")
					button.func = func
					button.a1 = info.arg1
					button.a2 = info.arg2
					button.a3 = info.arg3
					button.self = self
					button:SetScript("OnMouseUp", button_OnMouseUp)
					button:SetScript("OnMouseDown", button_OnMouseDown)
					button:SetScript("OnClick", button_OnClick)
					if button.clicked then
						button:SetButtonState("PUSHED")
					end
					button:EnableMouse(true)
				else
					button:SetScript("OnMouseDown", nil)
					button:SetScript("OnMouseUp", nil)
					button:SetScript("OnClick", nil)
					button:EnableMouse(false)
				end
			else
				button:SetScript("OnMouseDown", nil)
				button:SetScript("OnMouseUp", nil)
				button:SetScript("OnClick", nil)
				button:EnableMouse(false)
			end
		end
		tooltip.AddLine = wrap(tooltip.AddLine, "tooltip:AddLine")

		function tooltip:SetFontSizePercent(percent)
			local data, detachedData = self.data, self.detachedData
			if detachedData and detachedData.detached then
				data = detachedData
			end
			local lastSize = self.fontSizePercent
			percent = tonumber(percent) or 1
			if percent < 0.25 then
				percent = 0.25
			elseif percent > 4 then
				percent = 4
			end
			self.fontSizePercent = percent
			if data then
				data.fontSizePercent = percent
			end
			self.scrollUp:SetFont(font, normalSize * self.fontSizePercent, flags)
			self.scrollDown:SetFont(font, normalSize * self.fontSizePercent, flags)
			local ratio = self.fontSizePercent / lastSize
			for i = 1, self.numLines do
				local left = self.lefts[i]
				local right = self.rights[i]
				local third = self.thirds[i]
				local fourth = self.fourths[i]
				local fifth = self.fifths[i]
				local sixth = self.sixths[i]
				local check = self.checks[i]
				local font, size, flags = left:GetFont()
				left:SetFont(font, size * ratio, flags)
				font, size, flags = right:GetFont()
				right:SetFont(font, size * ratio, flags)
				font, size, flags = third:GetFont()
				third:SetFont(font, size * ratio, flags)
				font, size, flags = fourth:GetFont()
				fourth:SetFont(font, size * ratio, flags)
				font, size, flags = fifth:GetFont()
				fifth:SetFont(font, size * ratio, flags)
				font, size, flags = sixth:GetFont()
				sixth:SetFont(font, size * ratio, flags)
				check.width = check.width * ratio
				check:SetWidth(check:GetWidth() * ratio)
				check:SetHeight(check:GetHeight() * ratio)
			end
			self:SetWidth((self:GetWidth() - 51) * ratio + 51)
			self:SetHeight((self:GetHeight() - 51) * ratio + 51)
			if self:IsShown() and self.children then
				self:Show()
			end
		end
		tooltip.SetFontSizePercent = wrap(tooltip.SetFontSizePercent, "tooltip:SetFontSizePercent")

		function tooltip:GetFontSizePercent()
			return self.fontSizePercent
		end

		function tooltip:SetTransparency(alpha)
			local data, detachedData = self.data, self.detachedData
			if detachedData and detachedData.detached then
				data = detachedData
			end
			self.transparency = alpha
			if data then
				data.transparency = alpha ~= 0.75 and alpha or nil
			end
			self:SetBackdropColor(self.r or 0, self.g or 0, self.b or 0, alpha)
			self:SetBackdropBorderColor(1, 1, 1, alpha)
		end
		tooltip.SetTransparency = wrap(tooltip.SetTransparency, "tooltip:SetTransparency")

		function tooltip:GetTransparency()
			return self.transparency
		end

		function tooltip:SetColor(r, g, b)
			local data, detachedData = self.data, self.detachedData
			if detachedData and detachedData.detached then
				data = detachedData
			end
			self.r = r
			self.g = g
			self.b = b
			if data then
				data.r = r ~= 0 and r or nil
				data.g = g ~= 0 and g or nil
				data.b = b ~= 0 and b or nil
			end
			self:SetBackdropColor(r or 0, g or 0, b or 0, self.transparency)
			self:SetBackdropBorderColor(1, 1, 1, self.transparency)
		end
		tooltip.SetColor = wrap(tooltip.SetColor, "tooltip:SetColor")

		function tooltip:GetColor()
			return self.r, self.g, self.b
		end

		function tooltip:Scroll(down)
			if down then
				if IsShiftKeyDown() then
					self.scroll = self.numLines - GetMaxLinesPerScreen(self)
				else
					self.scroll = self.scroll + 3
				end
			else
				if IsShiftKeyDown() then
					self.scroll = 0
				else
					self.scroll = self.scroll - 3
				end
			end
			if self.scroll > self.numLines - GetMaxLinesPerScreen(self) then
				self.scroll = self.numLines - GetMaxLinesPerScreen(self)
			end
			if self.scroll < 0 then
				self.scroll = 0
			end
			if self:IsShown() then
				self:Show()
			end
		end
		tooltip.Scroll = wrap(tooltip.Scroll, "tooltip:Scroll")

		function tooltip.Detach(tooltip)
			local owner = tooltip.owner
			tooltip:Hide()
			self:assert(tooltip.detachedData, "You cannot detach if detachedData is not present")
			tooltip.detachedData.detached = true
			local detached = AcquireDetachedFrame(self, tooltip.registration, tooltip.data, tooltip.detachedData)

			detached.menu, tooltip.menu = tooltip.menu, nil
			detached.children = tooltip.children
			tooltip.children = nil
			detached:SetOwner(owner)
			detached:children()
			detached:Show()
		end
		tooltip.Detach = wrap(tooltip.Detach, "tooltip:Detach")

	end

	tooltip.registration = registration
	registration.tooltip = tooltip
	return tooltip
end
AcquireFrame = wrap(AcquireFrame, "AcquireFrame")

function ReleaseDetachedFrame(self, data, detachedData)
	if not detachedData then
		detachedData = data
	end
	for _, detached in ipairs(detachedTooltips) do
		if detached.detachedData == detachedData then
			detached.notInUse = true
			detached:Hide()
			detached.registration.tooltip = nil
			detached.registration = nil
		end
	end
end
ReleaseDetachedFrame = wrap(ReleaseDetachedFrame, "ReleaseDetachedFrame")

local StartCheckingAlt, StopCheckingAlt
do
	local frame
	function StartCheckingAlt(func)
		if not frame then
			frame = CreateFrame("Frame")
		end
		local last = IsAltKeyDown()
		frame:SetScript("OnUpdate", function()
			local now = IsAltKeyDown()
			if last ~= now then
				last = now
				func()
			end
		end)
	end
	function StopCheckingAlt()
		if frame then
			frame:SetScript("OnUpdate", nil)
		end
	end
end

function AcquireDetachedFrame(self, registration, data, detachedData)
	if not detachedData then
		detachedData = data
	end
	for _, detached in ipairs(detachedTooltips) do
		if detached.notInUse then
			detached.data = data
			detached.detachedData = detachedData
			detached.notInUse = nil
			local fontSizePercent = detachedData.fontSizePercent or 1
			local transparency = detachedData.transparency or 0.75
			local r = detachedData.r or 0
			local g = detachedData.g or 0
			local b = detachedData.b or 0
			detached:SetFontSizePercent(fontSizePercent)
			detached:SetTransparency(transparency)
			detached:SetColor(r, g, b)
			detached:ClearAllPoints()
			detached:SetPoint(detachedData.anchor or "CENTER", UIParent, detachedData.anchor or "CENTER", detachedData.offsetx or 0, detachedData.offsety or 0)
			detached.registration = registration
			registration.tooltip = detached
			return detached
		end
	end

	if not Dewdrop and AceLibrary:HasInstance("Dewdrop-2.0") then
		Dewdrop = AceLibrary("Dewdrop-2.0")
	end
	StartCheckingAlt(function()
		for _, detached in ipairs(detachedTooltips) do
			if detached:IsShown() and detached.locked then
				detached:EnableMouse(IsAltKeyDown())
				detached:children()
				if detached.moving then
					local a1 = arg1
					arg1 = "LeftButton"
					if type(detached:GetScript("OnMouseUp")) == "function" then
						detached:GetScript("OnMouseUp")()
					end
					arg1 = a1
				end
			end
		end
	end)
	if not tooltip then
		AcquireFrame(self, {})
	end
	local detached = CreateFrame("Frame", "Tablet20DetachedFrame" .. (table.getn(detachedTooltips) + 1), UIParent)
	table.insert(detachedTooltips, detached)
	detached.notInUse = true
	detached:EnableMouse(not data.locked)
	detached:EnableMouseWheel(true)
	detached:SetMovable(true)
	detached:SetPoint(data.anchor or "CENTER", UIParent, data.anchor or "CENTER", data.offsetx or 0, data.offsety or 0)

	detached.numLines = 0
	detached.owner = nil
	detached.fontSizePercent = 1
	detached.maxLines = 0
	detached.buttons = {}
	detached.checks = {}
	detached.lefts = {}
	detached.rights = {}
	detached.thirds = {}
	detached.fourths = {}
	detached.fifths = {}
	detached.sixths = {}
	detached.transparency = 0.75
	detached.r = 0
	detached.g = 0
	detached.b = 0
	detached:SetFrameStrata("BACKGROUND")
	detached:SetBackdrop(tmp.a(
		'bgFile', "Interface\\Buttons\\WHITE8X8",
		'edgeFile', "Interface\\Tooltips\\UI-Tooltip-Border",
		'tile', true,
		'tileSize', 16,
		'edgeSize', 16,
		'insets', tmp.b(
			'left', 5,
			'right', 5,
			'top', 5,
			'bottom', 5
		)
	))
	detached.locked = detachedData.locked
	detached.scroll = 0
	detached:EnableMouse(not detached.locked)

	local width = GetScreenWidth()
	local height = GetScreenHeight()
	detached:SetScript("OnMouseDown", function()
		if arg1 == "LeftButton" then
			detached:StartMoving()
			detached.moving = true
		end
	end)

	detached:SetScript("OnMouseUp", function()
		if arg1 == "LeftButton" then
			detached:StopMovingOrSizing()
			detached.moving = nil
			local anchor
			local offsetx
			local offsety
			if detached:GetTop() + detached:GetBottom() < height then
				anchor = "BOTTOM"
				offsety = detached:GetBottom()
				if offsety < 0 then
					offsety = 0
				end
				if offsety < MainMenuBar:GetTop() and MainMenuBar:IsVisible() then
					offsety = MainMenuBar:GetTop()
				end
				local top = 0
				if FuBar then
					for i = 1, FuBar:GetNumPanels() do
						local panel = FuBar:GetPanel(i)
						if panel:GetAttachPoint() == "BOTTOM" then
							if panel.frame:GetTop() > top then
								top = panel.frame:GetTop()
								break
							end
						end
					end
				end
				if offsety < top then
					offsety = top
				end
			else
				anchor = "TOP"
				offsety = detached:GetTop() - height
				if offsety > 0 then
					offsety = 0
				end
				local bottom = GetScreenHeight()
				if FuBar then
					for i = 1, FuBar:GetNumPanels() do
						local panel = FuBar:GetPanel(i)
						if panel:GetAttachPoint() == "TOP" then
							if panel.frame:GetBottom() < bottom then
								bottom = panel.frame:GetBottom()
								break
							end
						end
					end
				end
				bottom = bottom - GetScreenHeight()
				if offsety > bottom then
					offsety = bottom
				end
			end
			if detached:GetLeft() + detached:GetRight() < width * 2 / 3 then
				anchor = anchor .. "LEFT"
				offsetx = detached:GetLeft()
				if offsetx < 0 then
					offsetx = 0
				end
			elseif detached:GetLeft() + detached:GetRight() < width * 4 / 3 then
				if anchor == "" then
					anchor = "CENTER"
				end
				offsetx = (detached:GetLeft() + detached:GetRight() - GetScreenWidth()) / 2
			else
				anchor = anchor .. "RIGHT"
				offsetx = detached:GetRight() - width
				if offsetx > 0 then
					offsetx = 0
				end
			end
			detached:ClearAllPoints()
			detached:SetPoint(anchor, UIParent, anchor, offsetx, offsety)
			local t = detached.detachedData
			if t.anchor ~= anchor or math.abs(t.offsetx - offsetx) > 8 or math.abs(t.offsety - offsety) > 8 then
				detached.preventClick = GetTime() + 0.05
			end
			t.anchor = anchor
			t.offsetx = offsetx
			t.offsety = offsety
			detached:Show()
		end
	end)

	Dewdrop:Register(detached,
		'children', function(level, value)
			if not detached.registration then
				return
			end
			if detached.menu then
				detached.menu(level, value)
				if level == 1 then
					Dewdrop:AddLine()
				end
			end
			if level == 1 then
				if not detached.registration.cantAttach then
					Dewdrop:AddLine(
						'text', DETACH,
						'tooltipTitle', DETACH,
						'tooltipText', DETACH_DESC,
						'checked', true,
						'arg1', detached,
						'func', "Attach",
						'closeWhenClicked', true
					)
				end
				Dewdrop:AddLine(
					'text', LOCK,
					'tooltipTitle', LOCK,
					'tooltipText', LOCK_DESC,
					'checked', detached:IsLocked(),
					'arg1', detached,
					'func', "Lock",
					'closeWhenClicked', not detached:IsLocked()
				)
				Dewdrop:AddLine(
					'text', COLOR,
					'tooltipTitle', COLOR,
					'tooltipText', COLOR_DESC,
					'hasColorSwatch', true,
					'r', detached.r,
					'g', detached.g,
					'b', detached.b,
					'hasOpacity', true,
					'opacity', detached.transparency,
					'colorFunc', function(r, g, b, a)
						detached:SetColor(r, g, b)
						detached:SetTransparency(a)
					end
				)
				Dewdrop:AddLine(
					'text', SIZE,
					'tooltipTitle', SIZE,
					'tooltipText', SIZE_DESC,
					'hasArrow', true,
					'hasSlider', true,
					'sliderFunc', function(value)
						detached:SetFontSizePercent(value)
					end,
					'sliderMax', 2,
					'sliderMin', 0.5,
					'sliderStep', 0.05,
					'sliderIsPercent', true,
					'sliderValue', detached:GetFontSizePercent()
				)
				Dewdrop:AddLine(
					'text', CLOSE_MENU,
					'tooltipTitle', CLOSE_MENU,
					'tooltipText', CLOSE_MENU_DESC,
					'func', function()
						Dewdrop:Close()
					end
				)
			end
		end,
		'point', function()
			local x, y = detached:GetCenter()
			if x < GetScreenWidth() / 2 then
				if y < GetScreenHeight() / 2 then
					return "BOTTOMLEFT", "BOTTOMRIGHT"
				else
					return "TOPLEFT", "TOPRIGHT"
				end
			else
				if y < GetScreenHeight() / 2 then
					return "BOTTOMRIGHT", "BOTTOMLEFT"
				else
					return "TOPRIGHT", "TOPLEFT"
				end
			end
		end
	)

	NewLine(detached)

	detached.scrollUp = detached:CreateFontString(nil, "ARTWORK")
	detached.scrollUp:SetPoint("TOPLEFT", detached.buttons[1], "BOTTOMLEFT", 0, -2)
	detached.scrollUp:SetPoint("RIGHT", detached, "RIGHT", 0, -10)
	detached.scrollUp:SetFontObject(GameTooltipText)
	detached.scrollUp:Hide()
	local font,_,flags = detached.scrollUp:GetFont()
	detached.scrollUp:SetFont(font, normalSize * detached.fontSizePercent, flags)
	detached.scrollUp:SetJustifyH("CENTER")
	detached.scrollUp:SetTextColor(1, 0.823529, 0)
	detached.scrollUp:SetText(" ")

	detached.scrollDown = detached:CreateFontString(nil, "ARTWORK")
	detached.scrollDown:SetPoint("TOPLEFT", detached.buttons[1], "BOTTOMLEFT", 0, -2)
	detached.scrollDown:SetPoint("RIGHT", detached, "RIGHT", 0, -10)
	detached.scrollDown:SetFontObject(GameTooltipText)
	detached.scrollDown:Hide()
	local font,_,flags = detached.scrollUp:GetFont()
	detached.scrollDown:SetFont(font, normalSize * detached.fontSizePercent, flags)
	detached.scrollDown:SetJustifyH("CENTER")
	detached.scrollDown:SetTextColor(1, 0.823529, 0)
	detached.scrollDown:SetText(" ")

	detached:SetScript("OnMouseWheel", function()
		detached:Scroll(arg1 < 0)
	end)

	detached.SetTransparency = tooltip.SetTransparency
	detached.GetTransparency = tooltip.GetTransparency
	detached.SetColor = tooltip.SetColor
	detached.GetColor = tooltip.GetColor
	detached.SetFontSizePercent = tooltip.SetFontSizePercent
	detached.GetFontSizePercent = tooltip.GetFontSizePercent
	detached.SetOwner = tooltip.SetOwner
	detached.IsOwned = tooltip.IsOwned
	detached.ClearLines = tooltip.ClearLines
	detached.NumLines = tooltip.NumLines
	detached.__Hide = detached.Hide
	detached.__Show = detached.Show
	detached.Hide = tooltip.Hide
	detached.Show = tooltip.Show
	local old_IsShown = detached.IsShown
	function detached:IsShown()
		if self.tmpHidden then
			return true
		else
			return old_IsShown(self)
		end
	end
	detached.AddLine = tooltip.AddLine
	detached.Scroll = tooltip.Scroll
	function detached:IsLocked()
		return self.locked
	end
	function detached:Lock()
		self:EnableMouse(self.locked)
		self.locked = not self.locked
		self.detachedData.locked = self.locked or nil
		self:children()
	end

	function detached.Attach(detached)
		self:assert(detached, "Detached tooltip not given.")
		self:assert(detached.AddLine, "detached argument not a Tooltip.")
		self:assert(detached.owner, "Detached tooltip has no owner.")
		self:assert(not detached.notInUse, "Detached tooltip not in use.")
		detached.menu = nil
		detached.detachedData.detached = nil
		detached:SetOwner(nil)
		detached.notInUse = true
	end

	return AcquireDetachedFrame(self, registration, data, detachedData)
end
AcquireDetachedFrame = wrap(AcquireDetachedFrame, "AcquireDetachedFrame")

function Tablet:Close(parent)
	if not parent then
		if tooltip and tooltip:IsShown() then
			tooltip:Hide()
			tooltip.registration.tooltip = nil
			tooltip.registration = nil
			tooltip.enteredFrame = false
		end
		return
	else
		self:argCheck(parent, 2, "table", "string")
	end
	local info = self.registry[parent]
	self:assert(info, "You cannot close a tablet with an unregistered parent frame.")
	local data = info.data
	local detachedData = info.detachedData
	if detachedData and detachedData.detached then
		ReleaseDetachedFrame(self, data, detachedData)
	elseif tooltip.data == data then
		tooltip:Hide()
		tooltip.registration.tooltip = nil
		tooltip.registration = nil
	end
	tooltip.enteredFrame = false
end
Tablet.Close = wrap(Tablet.Close, "Tablet:Close")

local currentFrame
local currentTabletData

function Tablet:Open(fakeParent, parent)
	self:argCheck(fakeParent, 2, "table", "string")
	self:argCheck(parent, 3, "nil", "table", "string")
	if not parent then
		parent = fakeParent
	end
	local info = self.registry[parent]
	self:assert(info, "You cannot open a tablet with an unregistered parent frame.")
	self:Close()
	local data = info.data
	local detachedData = info.detachedData
	local children = info.children
	if not children then
		return
	end
	local frame = AcquireFrame(self, info, data, detachedData)
	frame.clickable = info.clickable
	frame.menu = info.menu
	local children = info.children
	function frame:children()
		if not self.preventRefresh then
			currentFrame = self
			currentTabletData = TabletData:new(self)
			self:ClearLines()
			if children then
				children()
			end
			currentTabletData:Display(currentFrame)
			self:Show(currentTabletData)
			currentTabletData:del()
			currentTabletData = nil
			currentFrame = nil
		end
	end
	frame:SetOwner(fakeParent)
	frame:children()
	local point = info.point
	local relativePoint = info.relativePoint
	if type(point) == "function" then
		local b
		point, b = point(fakeParent)
		if b then
			relativePoint = b
		end
	end
	if type(relativePoint) == "function" then
		relativePoint = relativePoint(fakeParent)
	end
	if not point then
		point = "CENTER"
	end
	if not relativePoint then
		relativePoint = point
	end
	frame:ClearAllPoints()
	if type(parent) ~= "string" then
		frame:SetPoint(point, fakeParent, relativePoint)
	end
	local offsetx = 0
	local offsety = 0
	if frame:GetBottom() and frame:GetLeft() then
		if frame:GetRight() > GetScreenWidth() then
			offsetx = frame:GetRight() - GetScreenWidth()
		elseif frame:GetLeft() < 0 then
			offsetx = -frame:GetLeft()
		end
		local ratio = GetScreenWidth() / GetScreenHeight()
		if ratio >= 2.4 and frame:GetRight() > GetScreenWidth() / 2 and frame:GetLeft() < GetScreenWidth() / 2 then
			if frame:GetCenter() < GetScreenWidth() / 2 then
				offsetx = frame:GetRight() - GetScreenWidth() / 2
			else
				offsetx = frame:GetLeft() - GetScreenWidth() / 2
			end
		end
		if frame:GetBottom() < 0 then
			offsety = frame:GetBottom()
		elseif frame:GetTop() and frame:GetTop() > GetScreenHeight() then
			offsety = frame:GetTop() - GetScreenHeight()
		end
		if MainMenuBar:IsVisible() and frame:GetBottom() < MainMenuBar:GetTop() and offsety < frame:GetBottom() - MainMenuBar:GetTop() then
			offsety = frame:GetBottom() - MainMenuBar:GetTop()
		end

		if FuBar then
			local top = 0
			if FuBar then
				for i = 1, FuBar:GetNumPanels() do
					local panel = FuBar:GetPanel(i)
					if panel:GetAttachPoint() == "BOTTOM" then
						if panel.frame:GetTop() and panel.frame:GetTop() > top then
							top = panel.frame:GetTop()
							break
						end
					end
				end
			end
			if frame:GetBottom() < top and offsety < frame:GetBottom() - top then
				offsety = frame:GetBottom() - top
			end
			local bottom = GetScreenHeight()
			if FuBar then
				for i = 1, FuBar:GetNumPanels() do
					local panel = FuBar:GetPanel(i)
					if panel:GetAttachPoint() == "TOP" then
						if panel.frame:GetBottom() and panel.frame:GetBottom() < bottom then
							bottom = panel.frame:GetBottom()
							break
						end
					end
				end
			end
			if frame:GetTop() > bottom and offsety < frame:GetTop() - bottom then
				offsety = frame:GetTop() - bottom
			end
		end
	end
	if type(fakeParent) ~= "string" then
		frame:SetPoint(point, fakeParent, relativePoint, -offsetx, -offsety)
	end

	if detachedData and (info.cantAttach or detachedData.detached) and frame == tooltip then
		detachedData.detached = false
		frame:Detach()
	end
	if (not detachedData or not detachedData.detached) and GetMouseFocus() == fakeParent then
		self.tooltip.enteredFrame = true
	end
end
Tablet.Open = wrap(Tablet.Open, "Tablet:Open")

function Tablet:Register(parent, k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20, k21, v21, k22, v22, k23, v23, k24, v24, k25, v25, k26, v26, k27, v27, k28, v28, k29, v29, k30, v30)
	self:argCheck(parent, 2, "table", "string")
	if self.registry[parent] then
		self:Unregister(parent)
	end
	local info
	if type(k1) == "table" and k1[0] then
		self:assert(type(self.registry[k1]) == "table", "Other parent not registered")
		info = copy(self.registry[k1])
		if type(v1) == "function" then
			info.point = v1
			info.relativePoint = nil
		end
	else
		info = new(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20, k21, v21, k22, v22, k23, v23, k24, v24, k25, v25, k26, v26, k27, v27, k28, v28, k29, v29, k30, v30)
	end
	self.registry[parent] = info
	info.data = info.data or info.detachedData or {}
	info.detachedData = info.detachedData or info.data
	local data = info.data
	local detachedData = info.detachedData
	if not self.onceRegistered[parent] and type(parent) == "table" and type(parent.SetScript) == "function" and not info.dontHook then
		if not Dewdrop then
			Dewdrop = AceLibrary("Dewdrop-2.0")
		end
		local script = parent:GetScript("OnEnter")
		parent:SetScript("OnEnter", function()
			if script then
				script()
			end
			if self.registry[parent] then
				if (not data or not detachedData.detached) and not Dewdrop:IsOpen(parent) then
					self:Open(parent)
					self.tooltip.enteredFrame = true
				end
			end
		end)
		local script = parent:GetScript("OnLeave")
		parent:SetScript("OnLeave", function()
			if script then
				script()
			end
			if self.registry[parent] then
				if self.tooltip and (not data or not detachedData or not detachedData.detached) then
					self.tooltip.enteredFrame = false
				end
			end
		end)
		if parent:HasScript("OnMouseDown") then
			local script = parent:GetScript("OnMouseDown")
			parent:SetScript("OnMouseDown", function()
				if script then
					script()
				end
				if self.registry[parent] and self.registry[parent].tooltip and self.registry[parent].tooltip == self.tooltip then
					self.tooltip:Hide()
				end
			end)
		end
		if parent:HasScript("OnMouseWheel") then
			local script = parent:GetScript("OnMouseWheel")
			parent:SetScript("OnMouseWheel", function()
				if script then
					script()
				end
				if self.registry[parent] and self.registry[parent].tooltip then
					self.registry[parent].tooltip:Scroll(arg1 < 0)
				end
			end)
		end
	end
	self.onceRegistered[parent] = true
	if GetMouseFocus() == parent then
		self:Open(parent)
	end
end
Tablet.Register = wrap(Tablet.Register, "Tablet:Register")

function Tablet:Unregister(parent)
	self:argCheck(parent, 2, "table", "string")
	self:assert(self.registry[parent], "You cannot unregister a parent frame if it has not been registered already.")
	self.registry[parent] = nil
end
Tablet.Unregister = wrap(Tablet.Unregister, "Tablet:Unregister")

function Tablet:IsRegistered(parent)
	self:argCheck(parent, 2, "table", "string")
	return self.registry[parent] and true
end
Tablet.IsRegistered = wrap(Tablet.IsRegistered, "Tablet:IsRegistered")

local _id = 0
local addedCategory
local currentCategoryInfo
local depth = 0
local categoryPool = {}
function CleanCategoryPool(self)
	for k,v in pairs(categoryPool) do
		del(v)
		categoryPool[k] = nil
	end
	_id = 0
end

function Tablet:AddCategory(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20, k21, v21, k22, v22, k23, v23, k24, v24, k25, v25, k26, v26, k27, v27, k28, v28, k29, v29, k30, v30)
	self:assert(currentFrame, "You must add categories in within a registration.")
	local info = new(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10, k11, v11, k12, v12, k13, v13, k14, v14, k15, v15, k16, v16, k17, v17, k18, v18, k19, v19, k20, v20, k21, v21, k22, v22, k23, v23, k24, v24, k25, v25, k26, v26, k27, v27, k28, v28, k29, v29, k30, v30)
	local cat = currentTabletData:AddCategory(info)
	del(info)
	return cat
end
Tablet.AddCategory = wrap(Tablet.AddCategory, "Tablet:AddCategory")

function Tablet:SetHint(text)
	self:assert(currentFrame, "You must set hint within a registration.")
	self:assert(not currentCategoryInfo, "You cannot set hint in a category.")
	currentTabletData:SetHint(text)
end
Tablet.SetHint = wrap(Tablet.SetHint, "Tablet:SetHint")

function Tablet:SetTitle(text)
	self:assert(currentFrame, "You must set title within a registration")
	self:assert(not currentCategoryInfo, "You cannot set title in a category.")
	currentTabletData:SetTitle(text)
end
Tablet.SetTitle = wrap(Tablet.SetTitle, "Tablet:SetTitle")

function Tablet:SetTitleColor(r, g, b)
	self:assert(currentFrame, "You must set title color within a registration")
	self:assert(not currentCategoryInfo, "You cannot set title color in a category.")
	self:argCheck(r, 2, "number")
	self:argCheck(g, 3, "number")
	self:argCheck(b, 4, "number")
	currentTabletData:SetTitleColor(r, g, b)
end
Tablet.SetTitleColor = wrap(Tablet.SetTitleColor, "Tablet:SetTitleColor")

function Tablet:GetNormalFontSize()
	return normalSize
end

function Tablet:GetHeaderFontSize()
	return headerSize
end

function Tablet:GetNormalFontObject()
	return GameTooltipText
end

function Tablet:GetHeaderFontObject()
	return GameTooltipHeaderText
end

function Tablet:SetFontSizePercent(parent, percent)
	self:argCheck(parent, 2, "table", "string")
	local info = self.registry[parent]
	if info then
		if info.tooltip then
			info.tooltip:SetFontSizePercent(percent)
		else
			local data = info.data
			local detachedData = info.detachedData
			if detachedData.detached then
				detachedData.fontSizePercent = percent
			else
				data.fontSizePercent = percent
			end
		end
	elseif type(parent) == "table" then
		parent.fontSizePercent = percent
	else
		self:assert(false, "You cannot change font size with an unregistered parent frame.")
	end
end
Tablet.SetFontSizePercent = wrap(Tablet.SetFontSizePercent, "Tablet:SetFontSizePercent")

function Tablet:GetFontSizePercent(parent)
	self:argCheck(parent, 2, "table", "string")
	local info = self.registry[parent]
	if info then
		local data = info.data
		local detachedData = info.detachedData
		if detachedData.detached then
			return detachedData.fontSizePercent or 1
		else
			return data.fontSizePercent or 1
		end
	elseif type(parent) == "table" then
		return parent.fontSizePercent or 1
	else
		self:assert(false, "You cannot check font size with an unregistered parent frame.")
	end
end
Tablet.GetFontSizePercent = wrap(Tablet.GetFontSizePercent, "Tablet:GetFontSizePercent")

function Tablet:SetTransparency(parent, percent)
	self:argCheck(parent, 2, "table", "string")
	local info = self.registry[parent]
	if info then
		if info.tooltip then
			info.tooltip:SetTransparency(percent)
		else
			local data = info.data
			local detachedData = info.detachedData
			if detachedData.detached then
				detachedData.transparency = percent
			elseif data then
				data.transparency = percent
			end
		end
	elseif type(parent) == "table" then
		parent.transparency = percent
	else
		self:assert(false, "You cannot change transparency with an unregistered parent frame.")
	end
end
Tablet.SetTransparency = wrap(Tablet.SetTransparency, "Tablet:SetTransparency")

function Tablet:GetTransparency(parent)
	self:argCheck(parent, 2, "table", "string")
	local info = self.registry[parent]
	if info then
		local data = info.data
		local detachedData = info.detachedData
		if detachedData.detached then
			return detachedData.transparency or 0.75
		else
			return data.transparency or 0.75
		end
	elseif type(parent) == "table" then
		return parent.transparency or 0.75
	else
		self:assert(parent, "You must provide a parent frame to check transparency")
	end
end
Tablet.GetTransparency = wrap(Tablet.GetTransparency, "Tablet:GetTransparency")

function Tablet:SetColor(parent, r, g, b)
	self:argCheck(parent, 2, "table", "string")
	local info = self.registry[parent]
	if info then
		if info.tooltip then
			info.tooltip:SetColor(r, g, b)
		else
			local data = info.data
			local detachedData = info.detachedData
			if detachedData.detached then
				detachedData.r = r
				detachedData.g = g
				detachedData.b = b
			else
				data.r = r
				data.g = g
				data.b = b
			end
		end
	elseif type(parent) == "table" then
		parent.r = r
		parent.g = g
		parent.b = b
	else
		self:assert(false, "You cannot change color with an unregistered parent frame.")
	end
end
Tablet.SetColor = wrap(Tablet.SetColor, "Tablet:SetColor")

function Tablet:GetColor(parent)
	self:argCheck(parent, 2, "table", "string")
	local info = self.registry[parent]
	if info then
		local data = info.data
		local detachedData = info.detachedData
		if detachedData.detached then
			return detachedData.r or 0, detachedData.g or 0, detachedData.b or 0
		else
			return data.r or 0, data.g or 0, data.b or 0
		end
	elseif type(parent) == "table" then
		return parent.r or 0, parent.g or 0, parent.b or 0
	else
		self:assert(parent, "You must provide a parent frame to check color")
	end
end
Tablet.GetColor = wrap(Tablet.GetColor, "Tablet:GetColor")

function Tablet:Detach(parent)
	self:argCheck(parent, 2, "table", "string")
	local info = self.registry[parent]
	self:assert(info, "You cannot detach tablet with an unregistered parent frame.")
	self:assert(info.detachedData, "You cannot detach tablet without a data field.")
	if info.tooltip and info.tooltip == tooltip then
		tooltip:Detach()
	else
		info.detachedData.detached = true
		local detached = AcquireDetachedFrame(self, info, info.data, info.detachedData)

		detached.menu = info.menu
		local children = info.children
		function detached:children()
			if not self.preventRefresh then
				currentFrame = self
				currentTabletData = TabletData:new(self)
				self:ClearLines()
				if children then
					children()
				end
				currentTabletData:Display(currentFrame)
				self:Show(currentTabletData)
				currentTabletData:del()
				currentTabletData = nil
				currentFrame = nil
			end
		end
		detached:SetOwner(parent)
		detached:children()
	end
end
Tablet.Detach = wrap(Tablet.Detach, "Tablet:Detach")

function Tablet:Attach(parent)
	self:argCheck(parent, 2, "table", "string")
	local info = self.registry[parent]
	self:assert(info, "You cannot detach tablet with an unregistered parent frame.")
	self:assert(info.detachedData, "You cannot attach tablet without a data field.")
	if info.tooltip and info.tooltip ~= tooltip then
		info.tooltip:Attach()
	else
		info.detachedData.detached = false
	end
end
Tablet.Attach = wrap(Tablet.Attach, "Tablet:Attach")

function Tablet:IsAttached(parent)
	self:argCheck(parent, 2, "table", "string")
	local info = self.registry[parent]
	self:assert(info, "You cannot check tablet with an unregistered parent frame.")
	return not info.detachedData or not info.detachedData.detached
end
Tablet.IsAttached = wrap(Tablet.IsAttached, "Tablet:IsAttached")

function Tablet:Refresh(parent)
	self:argCheck(parent, 2, "table", "string")
	local info = self.registry[parent]
	self:assert(info, "You cannot refresh tablet with an unregistered parent frame.")
	local tt = info.tooltip
	if tt and not tt.preventRefresh and tt:IsShown() then
		tt.updating = true
		tt:children()
		tt.updating = false
	end
end
Tablet.Refresh = wrap(Tablet.Refresh, "Tablet:Refresh")

function Tablet:IsLocked(parent)
	self:argCheck(parent, 2, "table", "string")
	local info = self.registry[parent]
	self:assert(info, "You cannot detach tablet with an unregistered parent frame.")
	return info.detachedData and info.detachedData.locked
end
Tablet.IsLocked = wrap(Tablet.IsLocked, "Tablet:IsLocked")

function Tablet:ToggleLocked(parent)
	self:argCheck(parent, 2, "table", "string")
	local info = self.registry[parent]
	self:assert(info, "You cannot detach tablet with an unregistered parent frame.")
	if info.tooltip and info.tooltip ~= tooltip then
		info.tooltip:Lock()
	elseif info.detachedData then
		info.detachedData.locked = info.detachedData.locked
	end
end
Tablet.ToggleLocked = wrap(Tablet.ToggleLocked, "Tablet:ToggleLocked")

function Tablet:UpdateDetachedData(parent, detachedData)
	self:argCheck(parent, 2, "table", "string")
	local info = self.registry[parent]
	self:assert(info, "You cannot detach tablet with an unregistered parent frame.")
	self:argCheck(detachedData, 3, "table")
	if info.data == info.detachedData then
		info.data = detachedData
	end
	info.detachedData = detachedData
	if info.detachedData.detached then
		self:Detach(parent)
	elseif info.tooltip and info.tooltip.owner then
		self:Attach(parent)
	end
end
Tablet.UpdateDetachedData = wrap(Tablet.UpdateDetachedData, "Tablet:UpdateDetachedData")

if DEBUG then
	function Tablet:ListProfileInfo()
		local duration, times, memories = GetProfileInfo()
		self:assert(duration and times and memories)
		local t = new()
		for method in pairs(memories) do
			table.insert(t, method)
		end
		table.sort(t, function(alpha, bravo)
			if memories[alpha] ~= memories[bravo] then
				return memories[alpha] < memories[bravo]
			elseif times[alpha] ~= times[bravo] then
				return times[alpha] < times[bravo]
			else
				return alpha < bravo
			end
		end)
		local memory = 0
		local time = 0
		for _,method in ipairs(t) do
			DEFAULT_CHAT_FRAME:AddMessage(format("%s || %.3f s || %.3f%% || %d KiB", method, times[method], times[method] / duration * 100, memories[method]))
			memory = memory + memories[method]
			time = time + times[method]
		end
		DEFAULT_CHAT_FRAME:AddMessage(format("%s || %.3f s || %.3f%% || %d KiB", "Total", time, time / duration * 100, memory))
		table_setn(t, 0)
		del(t)
	end
	SLASH_TABLET1 = "/tablet"
	SLASH_TABLET2 = "/tabletlib"
	SlashCmdList["TABLET"] = function(msg)
		TabletLib:GetInstance(MAJOR_VERSION):ListProfileInfo()
	end
end

local function activate(self, oldLib, oldDeactivate)
	Tablet = self
	if oldLib then
		self.registry = oldLib.registry
		self.onceRegistered = oldLib.onceRegistered
		self.tooltip = oldLib.tooltip
	else
		self.registry = {}
		self.onceRegistered = {}
	end

	tooltip = self.tooltip

	if oldDeactivate then
		oldDeactivate(oldLib)
	end
end

local function deactivate(self)
	StopCheckingAlt()
end

AceLibrary:Register(Tablet, MAJOR_VERSION, MINOR_VERSION, activate, deactivate)
```

## locales/enUS.lua
```lua
--[[ $Id: enUS.lua 19032 2006-12-05 03:02:51Z rabbit $ ]]--
local L = AceLibrary("AceLocale-2.2"):new("AuldLangSyne")
L:RegisterTranslations("enUS", function()
	return {	
		["AuldLangSyne"] = true,
		["Options for AuldLangSyne."] = true,

		--options
		["data"] = true,
		["Random data-management functions"] = true,
		["clear"] = true,
		["Clear stored data"] = true,
		["dump"] = true,
		["Print stored data"] = true,
		["ctimport"] = true,
		["Import notes for this realm from CT_PlayerNotes"] = true,
		["backup"] = true,
		["Options for backing up and restoring your friends list"] = true,
		["save"] = true,
		["Take a snapshot of your current friends list"] = true,
		["load"] = true,
		["Restore a snapshot of your friends list, wiping out your current friends list"] = true,

		["show"] = true,
		["Choose what information to display"] = true,
		["Turns display of PVP ranks on and off."] = true,
		["PVP rank numbers"] = true,
		["Turns display of PVP rank numbers on and off."] = true,
		["Guild"] = true,
		["Turns display of guild membership on and off."] = true,
		["Colors"] = true,
		["Choose which elements to color"] = true,
		["Name"] = true,
		["Turns coloring names by gender on and off."] = true,
		["Class"] = true,
		["Turns coloring class names on and off."] = true,
		["Compact mode"] = true,
		["Turns compact friends list on and off."] = true,
		["Compact mode levels"] = true,
		["Turns display of levels in compact mode on and off."] = true,
		["Note in player tooltip"] = true,
		["Turns display of player notes in tooltips on and off."] = true,
		["Note on logon"] = true,
		["Turns display of player notes on logon on and off."] = true,
		["Note on /who"] = true,
		["Turns display of player notes when you /who them on and off."] = true,
		["Hide minimap icon"] = true,
		
		-- output text
		["Detected an older database version.  Upgrading!"] = true,
		[" day"] = true,
		[" hour"] = true,
		[" minute"] = true,
		["s"] = true,
		[" ago"] = true,
		["Friend: "] = true,
		["Guild: "] = true,
		["Ignore: "] = true,
		["This slot is currently empty."] = true,
		["Backed up %d friends to slot %d"] = true,
		["Editing note for %s"] = true,
		["Click to edit"] = true,
		["Edit note"] = true,
		["Confirm"] = true,
		["Cancel"] = true,
    
	}
end)
```

## locales/esES.lua
```lua
local L = AceLibrary("AceLocale-2.2"):new("AuldLangSyne")
L:RegisterTranslations("esES", function()
	return {	
		["AuldLangSyne"] = "AuldLangSyne",
		["Options for AuldLangSyne."] = "Opciones para AuldLangSyne.",

		--options
		["data"] = "datos",
		["Random data-management functions"] = "Funciones de gesti\195\179n aleatoria de datos",
		["clear"] = "limpiar",
		["Clear stored data"] = "Limpia los datos guardados",
		["dump"] = "Volcar",
		["Print stored data"] = "Muestra los datos guardados",
		["ctimport"] = "ctimport",
		["Import notes for this realm from CT_PlayerNotes"] = "Importa notas para este reino desde CT_PlayerNotes",
		["backup"] = "backup",
		["Options for backing up and restoring your friends list"] = "Opciones para hacer copias de seguridad y restaurar tu lista de amigos",
		["save"] = "guardar",
		["Take a snapshot of your current friends list"] = "Guarda tu lista de amigos actual",
		["load"] = "cargar",
		["Restore a snapshot of your friends list, wiping out your current friends list"] = "Recupera una versi\195\179n anterior de tu lista de amigos, eliminando la actual",

		["show"] = "mostrar",
		["Choose what information to display"] = "Elige qu\195\169 informaci\195\179n se mostrar\195\161",
		["Turns display of PVP ranks on and off."] = "Determina si se muestran los rangos JcJ.",
		["PVP rank numbers"] = "N\195\186meros de rango JcJ",
		["Turns display of PVP rank numbers on and off."] = "Determina si se muestran los n\195\186meros de rango JcJ",
		["Guild"] = "Hermandad",
		["Turns display of guild membership on and off."] = "Determina si se muestran las Hermandades de tus amigos.",
		["Colors"] = "Colores",
		["Choose which elements to color"] = "Elige qu\195\169 elementos quieres colorear",
		["Name"] = "Nombre",
		["Turns coloring names by gender on and off."] = "Determina si se colorean los nombres por g\195\169nero.",
		["Class"] = "Clase",
		["Turns coloring class names on and off."] = "Determina si se colorean los nombres por clase.",
		["Compact mode"] = "Modo compacto",
		["Turns compact friends list on and off."] = "Determina si se compacta tu lista de amigos.",
		["Compact mode levels"] = "Niveles en modo compacto",
		["Turns display of levels in compact mode on and off."] = "Determina si se muestran los niveles en el modo compacto",
		["Note in player tooltip"] = "Nota en el tooltip del jugador",
		["Turns display of player notes in tooltips on and off."] = "Determina si se muestran las notas en los tooltip de los jugadores",
		["Note on logon"] = "Nota al conectar",
		["Turns display of player notes on logon on and off."] = "Determina si se muestra la nota de un jugador cuando se conecta",
		["Note on /who"] = "Nota en /who",
		["Turns display of player notes when you /who them on and off."] = "Determina si se muestra la nota de jugador cuando haces /who en ellos",
		["Hide minimap icon"] = true,
		
		-- output text
		["Detected an older database version.  Upgrading!"] = "Detectada una versi\195\179n de la base de datos anterior. \194\161Actualizando!",
		[" day"] = " d\195\173a",
		[" hour"] = " hora",
		[" minute"] = " minuto",
		["s"] = "s",
		[" ago"] = " atr\195\161s",
		["Friend: "] = "Amigo/a: ",
		["Guild: "] = "Hermandad: ",
		["Ignore: "] = "Ignorar: ",
		["This slot is currently empty."] = "Esta casilla est\195\161 actualmente vac\195\173a.",
		["Backed up %d friends to slot %d"] = "Copia de seguridad de %d amigos guardada en la casilla %d",
		["Editing note for %s"] = "Editando la nota de %s",
		["Click to edit"] = "Clic para editar",
		["Edit note"] = "Editar nota",
		["Confirm"] = "Confirmar",
		["Cancel"] = "Cancelar",
    
	}
end)
```

## locales/koKR.lua
```lua
--[[ $Id: koKR.lua 19032 2006-12-05 03:02:51Z rabbit $ ]]--
local L = AceLibrary("AceLocale-2.2"):new("AuldLangSyne")

L:RegisterTranslations("koKR", function()
	return {
		["AuldLangSyne"] = "AuldLangSyne 친구목록",
		["Options for AuldLangSyne."] = "AuldLangSyne 친구목록에 대한 설정입니다.",

		--options
		["data"] = "데이터",
		["Random data-management functions"] = "여러가지 데이터 관리 기능입니다.",
		["clear"] = "삭제",
		["Clear stored data"] = "저장된 데이터를 삭제합니다.",
		["dump"] = "덤프",
		["Print stored data"] = "저장된 데이터를 출력합니다.",
		["ctimport"] = "CT 불러오기",
		["Import notes for this realm from CT_PlayerNotes"] = "CT_PlayerNotes 에서 이 서버에 대한 메모를 불러옵니다.",
		["backup"] = "백업",
		["Options for backing up and restoring your friends list"] = "백업과 친구 목록 저장에 대한 설정입니다.",
		["save"] = "저장",
		["Take a snapshot of your current friends list"] = "현재 친구 목록을 저장합니다.",
		["load"] = "불러오기",
		["Restore a snapshot of your friends list, wiping out your current friends list"] = "백업된 친구 목록을 복원합니다.",

		["show"] = "표시",
		["Choose what information to display"] = "표시할 정보를 선택합니다.",
		["Turns display of PVP ranks on and off."] = "PVP 계급 표시를 활성화 혹은 비활성화 합니다.",
		["PVP rank numbers"] = "PVP 계급 번호",
		["Turns display of PVP rank numbers on and off."] = "PVP 계급 번호 표시를 활성화 혹은 비활성화 합니다.",
		["Guild"] = "길드",
		["Turns display of guild membership on and off."] = "길드 표시를 활성화 혹은 비활성화 합니다.",
		["Colors"] = "색상",
		["Choose which elements to color"] = "색상 사항을 선택합니다.",
		["Name"] = "이름",
		["Turns coloring names by gender on and off."] = "성별에 대한 이름 색상화를 활성화 혹은 비활성화 합니다.",
		["Class"] = "직업",
		["Turns coloring class names on and off."] = "직업 색상화를 활성화 혹은 비활성화 합니다.",
		["Compact mode"] = "간략한 모드",
		["Turns compact friends list on and off."] = "간략한 친구 목록 기능을 활성화 혹은 비활성화 합니다.",
		["Compact mode levels"] = "간략한 모드 레벨",
		["Turns display of levels in compact mode on and off."] = "간략한 모드에 레벨 표시를 활성화 혹은 비활성화 합니다.",
		["Note in player tooltip"] = "플레이어 툴팁 메모",
		["Turns display of player notes in tooltips on and off."] = "툴팁에 플레이어 메모를 표시하는 기능을 활성화 혹은 비활성화 합니다.",
		["Note on logon"] = "접속 메모",
		["Turns display of player notes on logon on and off."] = "접속시 플레이어 메모를 표시하는 기능을 활성화 혹은 비활성화 합니다.",
		["Note on /who"] = "/누구 메모",
		["Turns display of player notes when you /who them on and off."] = "/누구 실행 시 플레이어 메모 표시를 켜거나 끕니다.",
		["Hide minimap icon"] = true,
		
		-- output text
		["Detected an older database version.  Upgrading!"] = "이전 버전의 데이터베이스가 발견되었습니다. 갱신 중!",
		[" day"] = " 일",
		[" hour"] = " 시간",
		[" minute"] = " 분",
		["s"] = "",
		[" ago"] = " 전",
		["Friend: "] = "친구: ",
		["Guild: "] = "길드: ",
		["Ignore: "] = "제외: ",
		["This slot is currently empty."] = "해당 슬롯은 현재 비어있습니다.",
		["Backed up %d friends to slot %d"] = "%d명의 친구를 %d 슬롯으로 백업",
		["Editing note for %s"] = "%s 메모 수정",
		["Click to edit"] = "클릭 : 수정",
		["Edit note"] = "메모 편집",
		["Confirm"] = "확인",
		["Cancel"] = "취소",
    
	}
end)
```

## locales/zhCN.lua
```lua
--[[ $Id: zhCN.lua 19032 2006-12-05 03:02:51Z rabbit $ ]]--
local L = AceLibrary("AceLocale-2.2"):new("AuldLangSyne")
L:RegisterTranslations("zhCN", function()
	return {
--		["AuldLangSyne"] = true,
--		["Options for AuldLangSyne."] = true,

		--options
		["data"] = "数据管理",
		["Random data-management functions"] = "数据管理",
		["clear"] = "清除已保存的数据",
		["Clear stored data"] = "清除已保存的数据",
		["dump"] = "列出已保存的数据",
		["Print stored data"] = "列出已保存的数据",
		["ctimport"] = "导入CT_PlayerNotes数据",
		["Import notes for this realm from CT_PlayerNotes"] = "从CT_PlayerNotes导入本服务器的数据",
		["backup"] = "备份/恢复",
		["Options for backing up and restoring your friends list"] = "备份和恢复好友列表选项",
		["save"] = "保存快照",
		["Take a snapshot of your current friends list"] = "生成当前好友列表的快照备份",
		["load"] = "恢复快照",
		["Restore a snapshot of your friends list, wiping out your current friends list"] = "从一份好友备份快照中恢复, 将清除当前的好友列表",

		["show"] = "显示设置",
		["Choose what information to display"] = "选择要显示的信息",
		["Turns display of PVP ranks on and off."] = "显示PvP军衔名称",
		["PVP rank numbers"] = "显示PvP军衔等级",
		["Turns display of PVP rank numbers on and off."] = "显示PvP军衔等级",
		["Guild"] = "显示公会名称",
		["Turns display of guild membership on and off."] = "显示公会名称",
		["Colors"] = "显示彩色名字",
		["Choose which elements to color"] = "选择显示颜色方案",
		["Name"] = "按照性别",
		["Turns coloring names by gender on and off."] = "根据性别显示不同的颜色",
		["Class"] = "按照职业",
		["Turns coloring class names on and off."] = "根据职业显示不同的颜色",
		["Compact mode"] = "简洁模式",
		["Turns compact friends list on and off."] = "使用简洁模式显示好友列表",
		["Compact mode levels"] = "在简洁模式中显示等级",
		["Turns display of levels in compact mode on and off."] = "在简洁模式中显示等级",
		["Note in player tooltip"] = "信息提示上显示备注",
		["Turns display of player notes in tooltips on and off."] = "在信息提示上显示用户备注",
		["Note on logon"] = "登陆显示备注",
		["Turns display of player notes on logon on and off."] = "在好友登陆时显示用户备注",
		["Note on /who"] = "查询显示备注",
		["Turns display of player notes when you /who them on and off."] = "在查询好友时显示用户备注",
		["Hide minimap icon"] = true,

		-- output text
		["Detected an older database version.  Upgrading!"] = "检测到旧版本数据. 开始升级!",
		[" day"] = "日",
		[" hour"] = "小时",
		[" minute"] = "分钟",
		["s"] = "",
		[" ago"] = "以前",
		["Friend: "] = "好友: ",
		["Guild: "] = "公会: ",
		["Ignore: "] = "忽略: ",
		["This slot is currently empty."] = "该配置当前是空的.",
		["Backed up %d friends to slot %d"] = "已备份 %d 个好友到配置 %d",
		["Editing note for %s"] = "编辑 %s 的备注",
		["Click to edit"] = "点击编辑备注",
		["Edit note"] = "编辑备注",
		["Confirm"] = "确定",
		["Cancel"] = "取消",

	}
end)
```

## readme.md
```md
\"Should auld acquaintance be forgot, you'll need something to remember them by.\"

## Description:
**AuldLangSyne** combines the functions of CT_PlayerNotes and FriendsFacts.

It will remember information about friends and display it in your friends list while they are offline.

It also allows you to write short notes about these friends.

Type /auld for a (very minimal) command listing or use the FuBar icon.

If you already use CT_PlayerNotes, load the game with both addons enabled, and type "/auld ctimport" to load its notes.  (Due to the way CT_PlayerNotes stores its data, you will have to do this once for each character whose friends you have made notes on.)

### FAQ:
Q: WTF is the name AuldLangSyne, d00d?  
A: Well, Timmy, Auld Lang Syne is the name of a song, mostly sung around New Years Eve, whose opening line goes "should auld acquaintance be forgot, and never brought to mind."  (Literally, "auld lang syne' translates from Scots as "old long since", but may be better rendered as "times gone by".)  
In this, as in all things, Wikipedia shall be your guide: [Auld_Lang_Syne](http://en.wikipedia.org/wiki/Auld_Lang_Syne)  
I didn't expect this to be so little known.  ;_;

#### Credits:
*Original addon by Kemayo in 2006.* 
```

