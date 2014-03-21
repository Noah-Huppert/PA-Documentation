Updates made for 63180, but this will always be a work in progress.

## Index
- Super high level overview
- Overview of API calls
- Description of starting a game
- Data structures
- API methods
- Handlers Used

## Super high level overview:
- Request a server from UberNet
- Attach to the server and wait for a state change
- Configure the game
- Players join armies
- Wait for the server to be ready
- Start the game
- Wait for a state change

## Overview of API calls:

Heading is the scene in PA.

### connect_to_game:
- engine ubernet.startGame/ubernet.joinGame
- engine join_game
- handlers.login_accepted
- app.hello
- handlers.server_state payload.url

### new_game:
- send_message modify_settings with an object
- send_message modify_system with a server-formatted system
- send_message reset_armies with armies
- optionally: add_army, remove_army, leave_army, modify_army, next_primary_color, next_secondary_color, chat_message, update_commander, leave
- send_message join_army
- send_message toggle_ready
- handlers.control payload.sim_ready == true
- send_message start_game
- handlers.server_state payload.url

## Starting a Game

### Request a server from UberNet

A game requires a server.  The only servers we have available right now are UnberNet servers.  You can request a new server from UberNet, or request to join an existing instance.

    engine.asyncCall("ubernet.startGame", model.uberNetRegion(), 'Config').done(function (data) {
      data = JSON.parse(data);
      model.lobbyId(data.LobbyID);

      // continue
    }).fail(function (data) {
      console.log("failed to start ubernet game");
      reset();
    });
 
The method returns a promise with .done and .fail methods.  The payload object passed to the done handler provides a LobbyID which might be of use if you want to connect certain other clients to the same game.

If you are joining a game someone else has created, you will need the LobbyID.

    engine.asyncCall("ubernet.joinGame", lobbyId).done(function (data) {
      data = JSON.parse(data);

      if (data.PollWaitTimeMS) {
         // wait and then recurse
      } else {
        console.log("ubernet game join successful, will connect now");
        // continue
      }
    }).fail(function (data) {
      console.log("failed to join ubernet game");
      reset();
    });

The reset method calls one additional engine method, which is used to help ensure the game stays in a consistent state.

    engine.call('reset_game_state');

### Connect to a server

Once you've either created a server or registered your status with UberNet, you need to connect tot he server itself (presumably this is where a local server would pick up)

    engine.call('join_game',
      String(data.ServerHostname),
      Number(data.ServerPort),
      String(model.displayName()),  // the user's name
      String(data.Ticket),
      String(JSON.stringify({ password: undefined })));

After issuing this call, listen for a login_accepted event.  This is where alpha-ui shows the transmit message.

        app.hello(handlers.server_state, handlers.connection_disconnected);

listen to handlers.server_state for msg.state == 'lobby'

### Configuring the Game

Once you've joined the game, it must be configured.  See the data structures below for fields in a settings object and system. An important detail is that system format used by the client and server slightly different, and may need to be translated.

    model.send_message('modify_settings', settings, optionalCallback)
    model.send_message('modify_system', system, optionalCallback)

The army information was recently moved out of the game description to a number of detailed methods.  If you know exactly what you want to make, call reset_armies

    var armies = [
      { "slots" : 1, ai: false, alliance: false},
      { "slots" : 1, ai: true, alliance: false, economy_factor: -1 }
    ]

    model.send_message('reset_armies', armies, optionalCallback)

If you are less certain of the situation, you may also use add_army, modify_army, and remove_army.

### Players join armies

Once the game is configured, players may join armies.

    model.send_message('join_army', {
      army: slot, // army index
      commander: { ObjectName: model.preferredCommander().ObjectName }
    });

Once a player has joined an army (the methods accept no parameter to say otherwise), the player may call next_primary_color, next_secondary_color, or update_commander.  In theory chat_message is available as soon as you've joined a game.

### Start the game

Players need to send the toggle_ready message before the game may begin. The server must be in the sim_ready state, which can be observed on handlers.control

Once all players and the server are ready, the host may start the game by sending start_game.  Failure due to misconfiguration is common here, so make sure to report errors on this method.

    model.send_message('start_game', undefined, function(success) {
      if (!success) {
        console.log('start_game failed')
        reset()
      }
    });

All you can do from there is wait for the server_state message with payload.state == 'landing' and follow the payload.url.  If you are spectating, the state will be 'playing' instead.

## Data Structures

### Settings

  {
    "blocked" : [],
    "public": false,
    "friends" : [],
    "password" : undefined,
    "spectators" : 0,
    "type" : '0',
  }

Currently valid types are:

'0': Free for All
'1': Team Armies

Obsolete types were:

'2': Alliance
'3': Versus AI

### Army

    {
      slots: 1,
      ai: false,
      alliance: false,
      economy_factor: 1.0, // only valid for AI
    }

### System

    {
      "name": "Just a Small Moon",
      "planets": []
    }

### Planet

Server and client formats are slightly different, see the game code for conversion examples.

#### Server


        {
          "name": "Small Moon",
          "mass": 10000,
          "position": [x, y],
          "velocity": [x, y],
          "required_thrust_to_move": 0,
          "starting_planet": true,
          "generator": {
             "seed": 78462,
             "radius": 400,
             "heightRange": 75,
             "waterHeight": 0,
             "temperature": 0,
             "metalDensity": 50,
             "metalClusters": 50,
             "biomeScale": 50,
             "biome": "moon"
          }
        }


#### Client


        {
          "name": "Small Moon",
          "mass": 10000,
          "position_x": 53567.4,
          "position_y": 8786.55,
          "velocity_x": -15.5348,
          "velocity_y": 94.7081,
          "required_thrust_to_move": 0,
          "starting_planet": true,
          "planet": {
             "seed": 78462,
             "radius": 400,
             "heightRange": 75,
             "waterHeight": 0,
             "temperature": 0,
             "metalDensity": 50,
             "metalClusters": 50,
             "biomeScale": 50,
             "biome": "moon"
          }
        }


## API methods

### app.hello
arguments: success callback, failure callback

success callback is compatible with the server_state handler, and failure with connection_disconnected.  I can be a way to kickstart processes driven by server state.

### engine methods


#### join_game
arguments: hostname, port, user name, ticket, options(json string)
returns: none/unknown

known options (pre-string)

    {
       password:,
    }


#### reset_game_state
arguments: none
returns: none/unknown

#### ubernet.startGame
arguments: ubernet region, 'Config'
returns object:

    {
      LobbyID: ,
      ServerHostname:,
      ServerPort:,
      TIcket:,
      // possibly more
    }

#### ubernet.joinGame
arguments: lobby id
returns: object:

   {
     PollWaitTimeMS: ,
     // possibly more
   }


### send_message methods

#### add_army
arguments: {options: Army}

Host adding a new slot (or set of slots for team)

#### chat_message
arguments: {message: ""}

#### join_army
arguments: {army: index, commander: {ObjectName: ""}}

Player entering an empty slot in an existing army.

#### kick
arguments: {id: user_id}

#### leave
arguments: none

Leave the lobby.  Oposite of join_game.  alpha-ui then returns to start screen.

#### leave_army
arguments: none

Leave the currently assigned army and reopen the slot, but stay attached to the game.

#### modify_army
arguments {army_index: index, options: Army(subset)}

Allows setting of a single army property after creation.

#### modify_settings
arguments: Settings

Introduced in 62318, the game uses this message when settings other than system are changed, and sends it without system or enable_lan.

#### next_primary_color
#### next_secondary_color
arguments: none

#### remove_army
arguments: {army_index: 0}

Remove an entire army of slots from the game.  Alpha-ui always passes index 0.

#### reset_armies
arguments: [ Army ]

Completely replace the set of armies.  Used by alpha-ui when changing game types, but also the most straightforward method for API-centric applications.

#### set_loading
arguments: {loading: bool}

Set the player's loading indicator.

#### start_game
arguments: none

Initiates planet generation and eventually game play.  What happens when the host player clicks start.  Fails unless all other players are ready (toggle_ready)

#### toggle_ready
arguments: none

#### update_commander
arguments: {commander: {ObjectName: ""}}

Unlike colors, the protocol supports setting the commander directly; the cycling behavior is part of the alpha-ui.

The game provides a catalog of commanders where ObjectNames can be looked up, currently located in ui/alpha/shared/js/catalog.js

## Handlers used

### connection_disconnected
arguments: none

### connection_failed
arguments: none

### control
arguments: {sim_ready: bool, ...}

The server must be sim_ready before sending the start_game message.

### login_accepted
arguments: none

### login_rejected
arguments: none

### server_state
arguments: object:

   {
     state: '"",
     url: "",
     data: {
       armies: [Army],
       colors: [],
       control: {sim_ready: bool, ...},
       players: [],
       settings: {game_mode: Type},
       system: System,
     }
   }

Most of the data fields have dedicated messages that send only that piece of data.  control is the most imporant for successfully starting a game.
