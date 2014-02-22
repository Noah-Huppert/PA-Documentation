PlayerViewModel(name, commander, is_player)
	VARS
		name - Player name, set by input "name"
		commander - Commander selected by player in lobby, set by input "commander"
		isPlayer - *Determines if player object is AI* , set by input "is_player"
		commanderImageSource - Image for commander, determined by searching img folder of commander name


ArmyViewModel(players, slots, primary_color, secondary_color, player_display_name)
	VARS
		p - Array of players on army
		isAi - Boolean determining if player object is AI
		isPlayer - Boolean determining if player object is real player
		numSlots - Number of slots in the army
		armyLocal - Is player currently in this army
		players - p after being setup by above for statement
		numberOfSlots - Number of slots, setup by above numSlots
		openSlots - Amount of open slots set by, numberOfSlots - players().length
		primaryColor - Primary color shown for team
		secondaryColor - Secondary color shown for team
		readOnly - Is player allowed to make changes to team, set as oposite of local(If player on team readonly is false and vice versa)
		isAi - set as, isAi and oposite of isPlayer
	
	ACTIONS
		**for statement** iterating through input "slots"
			If current slot is a STRING set up name, commander, local, armyLocal and set isPlayer to TRUE
			If current slot is AI setup name, commander, local, and isAi to TRUE
			add onto player array with new PlayerViewModel(name, commander, local)


ChatMessageViewModel(name, type, payload)
	VARS
		username - Username of player who sent message, set as input "name"
		typeArray - Array of possible message types [invalid, enter, exit, lobby, whisper, global, team]
		typeIndex - Index of message type in typeArray, set for now as 0 and later set to actual type
		typeString - typeArray[typeIndex]
		payload - Actaul message contents, set as input "payload"

	ACTIONS
		**for statement iterating through typeArray**
			Sets typeIndex acording to input "type" by checking if typeArray[i] = type


LobbyViewModel()
	VARS
		showPopUp - Show popup element, a popup is for instance the leave lobby message
		popUpPrimaryMsg - Main message in popup, set as "Exit Game?"
		popupSecondaryMsg - Secondary message in popup, set as ""
		popupPrimaryButtonAction - Action that occurs when primary popup button is clicked, set to navagate to server browser
		popUpSecondaryButtonAction - Action that occurs when secondary popup button is clicked, set to hide popup
		popupTertiaryButtonAction - Action that occurs when tertiary popup button is clicked, set as hide popup
		popUpPrimaryButtonTag - Text in primary popup button, set as "Yes"
		popUpSecondaryButtonTag - Text in secondary popup button, set as "Cancel"
		popUpTertiaryButtonTag - Text in tertiary popup button, set as FALSE
		uberName - Ubernet username, set to Ubernet username
		displayName - Display name, set to steam name(If you have a steam key)
		uberId - Ubernet ID, set to Ubernet ID
		validCommanders - Array of valid commanders to select [delta, alpha, raptor, quad, tank]
		selectedCommanderIndex - Index of commander in validCommanders
		preferredCommander - Selected commander
		lobbyId - ID for id'ing lobby
		gameticket - gameTicket
		gameHostName - Server who hosts game, currently only servers are "Ubernet"
		gamePort - Game port(In IP)
		buildVersion - Build version of game, currently set  by what checkin number the build was made from on server(Total checkins, not just for PA)
		transitPrimaryMessage - transitPrimaryMessage
		transitSecondaryMessage - transitSecondaryMessage
		transitDestination - transitDestination
		transitDelay - transitDelay
		clientIsMod - Set by looping through roster and seeing if players own Ubernet ID is a mod
		isGameCreator - Boolean stating wether or not player is lobby creator, used in mod privalages
		createdGameDesc - Game description
		contacts - Player contacts
		roster - Players in lobby
		selectedRosterIndex - ASSUMED used for when you can perform actions on people in lobby
		chatSelected - Has user selected chat box, set to false
		serverLobbyReady - Is entire lobby ready, set to false
		serverPlanetsReady - Is server done generating planets, set to false
		showServerLoading - Is server currently loading planets, set to oposite of serverPlanetsReady
		showGameLoading - Show if game is loading, set to false
		showGameConfig - Show game config options, set by serverLobbyReady and the oposite of showGameLoading
		planetsAreReady - Shows if planets are ready, set to false
		waitingString - String shown after player has been set to ready, set to ""
		ownerName - String showing lobby owner name, set to ""
		teamTypes - Array holding team game types [Free For All, Team Armies, Alliance, Versus AI]
		showFFA - show FFA, set to false
		showTA - showTA, set to false
		showVA - Something to do with economie modifiers, set to false
		allowEconomyModifiers - Set to showVA
		serverPlayerState - Server player state, set to empty JSON data
		maxPlayers - Maxiumum players in loby, set to 0
		playerCountString - Shows amount of players in lobby, set to "(<Current Player Count> / <Max Player Count>)"
		serverArmyState - serverArmyState, set to empty array
		armies - Armies, set to empty array
		aiEconomyModifier - Eco modifier for AI, set to 1.0
		playerEconomyModifier - Eco modifier for player, set to 1.0
		armyListHeight - Height of army list element, set to 400
		armyListHeightString - String used in CSS of army list, set as "<height> px"
		hasToggledReady - Boolean indicating if player has clicked ready, set to false
		allowToggledReady - Boolean indicating if player can change its ready state, set to oposite of hasToggledReady(If player is ready player cannot un-ready and vise versa)
		loadedSystem - Loaded system for game session
		loadedSystemIsEmpty - Boolean indicating wether or not system is empty, set to true if loadedSystem is empty
		systemName - Name of loaded system, set to loadedSystem.name, if empty set to "Beta System"
		numPlanets - Number of planets in a system, set to loadedSystem.length, if system is empty set to 1
		primaryPlanet - Spawning planet in system, set to first planet, if system is empty set to {}
		loadedPlanet - Loaded planet
		planets - Array of planets
		loadedPlanetIsEmpty - Boolean indicating if loadedPlanet is empty, set to true if empty, set to false if not
		biomes - Name for biome types of planets [earth, lava, metal, moon, tropical]

	ACTIONS
		showKickPromoteUser - Set as
			FALSE if
				Player is not mod
				user doesnt exist
				user trying to kick/promote is a mod
			TRUE if
				anything else
		showLoadingReady - Set as
			0 if
				Both planets and server are not ready
			1 if
				Only planets or server is ready
			2 if
				both planets and server are ready
		armyListMaxHeight - Set as
			100 if
				there are 0 armies
			146 if
				there is 1 army
			192 if
				there are 2 armies
			238 if
				there are 3 armies
			285 if
				there are 4 armies
			285 if
				by defualt
		hasToggledReadyMsg - Set as
			"Game will start when the server has finished generating the solar system" if
				all players have toggled ready
			"Game will start when all players select a slot and click Ready." if
				all players are not ready

	FUNCTIONS
		maybeAddContact - function(contact)
			Determines if player can add player as contact(If they have friended them already or not)
			If player has tried to add player as contact is does
		kickUser - function(index)
			index - Player position in roster list
			Kicks user by sending a "kick" message, message contents in JSON { id: id }
		promoteUserToMod - function(index)
			index - Player position in roster list
			Promotes user by sending a "promote_to_mod" message, message contents in JSON { id: id }
		editing - function()
			Returns true if player input focus element is more than 0 and false if it is not
		resetPlayerPattern - function()
			Sets selectedPlayerIndex to 0
		navToServerBrowser - function()
			Call abandon - Defined bellow
			Call disconnect after 10 seconds then set view to server_browser.html
		selectedTeamTypeChanged - function()
			NOT IMPLIMENTED YET
		join - function(army_index, slot_index)
			army_index - Army number to join
			slot_index - Army slot to join CURRENTLY NOT IMPLIMENTED
			m - Message, set as emtpy array
			m.army - Army number to join, set as input "army_index"
			m.commander - Commander to set as when joining army, set as last selected commander(preferredCommander)
			Sends message "join_army" with message as m
		updateCommander - function()
			m - Message, set as empty array
			m.commander - New commander, set as selected commander(preferredCommander)
			Sends nessage "update_commander" with message as m
		changeCommander - function()
			Changed selectedCommanderIndex to be one greater than it is already
			Calls updateCommander
		updateEconomeyModifiers - function()
			message - Message, set as JSON { armies: [] }
			foreach army
				econRate - Player and AI economey rates
				add onto message(starting_metal_override: 2000.0, starting_energy_override: 2000.0, economy_rate_override: ecomRate)
			Sends message :update_economy_modifiers" with message message
		abandon - function()
			Send message "leave" with no message
			Makes engine asycnronous call "ubernet.removePlayerFromGame"
		toggle_ready - function()
			Checks if player is already ready
				if TRUE
					Sends "toggle_ready" message with message undefined and a callback that sets player status to ready if message succeeds
				if FALSE
					Nothing
		nextPrimaryColor - function()
			Sends message "next_primary_color" with no message
		nextSecondaryColor - function()
			Sends message "next_secondary_color" with no message
		update - function()
			sets waitingString to 3 "." then resets
		sendChat - function(message)
			message - Chat message text
			msg - Chat message to send with Message, set to empty array
			msg.message - Chat message, set to chat input box content
			Sends message "chat_message" with message msg if msg.message is not empty
			Clears chat inpu box content
		onResize - function()
			If window height it greater than 720px
				armyListHeight is set to window.innerHeight - 320
			Else
				armyListHeight is set to 400
		imageSourceForPlanet - function(planet)
			planet - Planet object that holds planet data
			ice - Boolean idicating if input "planet" is an ice planet, set to true if it has an earth biome and if its temperature is less than or equal to -0.5
			s - Biome type, set to "ice" if ice is true set to input "planet" biome
			S is then checked to make sure it has been set, if it has not been it is set to "unknown"
			Returns path to '../shared/img/' + s + '.png'
		planetSizeClass - function(size)
			size - Size of planet
			surfaceArea - Surface area of panet used for clasifing scale, set as 4* PI * size^2
			If surface area is
				<= 1000000 return '1'
				<= 6000000 return '2'
				<= 12000000 return '3'
				<= 18000000 return '4'
				<= 30000000 return '5'
				defualt return 'unknown'


		ENDED DOCS ON LINE 392





	




