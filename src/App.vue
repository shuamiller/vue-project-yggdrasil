<template>
  <div>
    <h1>
      <span class="header-span">p</span>roject<span class="header-span"
        >Ygg</span
      >drasil
    </h1>
    <p id="subtitle">A Textploration Game</p>
  </div>
  <div id="console">
    <div id="game-info">
      <div v-if="!gameStarted" id="welcome-div">
        <p id="welcome-message" class="game-text">Welcome to Yggdrasil</p>
        <p class="game-text">
          You will find yourself in a new Region distant from what you know.
          Discover ways to explore and interact with the Region. Colors will
          tell you whether you can interact with something and whether it is a
          <span class="character-text">Character</span>,
          <span class="object-text">Object</span>,
          <span class="location-text">Location</span>, or
          <span class="direction-text">Direction</span>.
        </p>
        <p class="game-text">
          Start off your actions with a <span class="verb-text">Verb</span>.
          Some things to try: <span class="verb-text">Examine</span>,
          <span class="verb-text">Walk</span>,
          <span class="verb-text">Take</span>.
        </p>
        <p class="game-text">
          It is important to remember there is no "win-state or plot goals.
          projectYggdrasil is focused on discovery and exploration by means of
          curiosity. Uncover rare storylines, locations, and characters. Most
          "progress" in this game is non-linear. There is no expected route or
          right way to play.
        </p>
        <p class="game-text">
          Lastly, if you need help, just ask <span class="verb-text">?</span>
        </p>
      </div>
      <div v-if="gameStarted" id="location-div">
        <div id="Region-location">
          <p id="Region-text" class="location-header-text">
            {{ currentRegionName }}
          </p>
        </div>
        <div id="room-location">
          <p id="room-text" class="location-header-text">
            {{ currentRoomName }}
          </p>
        </div>
      </div>
      <div v-if="gameStarted" id="description-div">
        <p
          v-html="highlightText(currentRoomDescription)"
          id="room-description-text"
          class="game-text"
        ></p>
      </div>
      <div v-if="gameStarted" id="action-response-div" ref="actionResponseDiv">
        <p
          v-html="highlightText(message)"
          v-for="(message, index) in messages"
          :key="index"
          class="game-text"
        ></p>
      </div>
    </div>
    <div id="player-interface">
      <div id="begin-div" v-if="!gameStarted">
        <button id="begin-btn" @click="startGame">Begin</button>
      </div>
      <div id="input-div">
        <input
          id="player-input"
          type="text"
          autocomplete="off"
          :placeholder="placeholder"
          @keyup.enter="checkInput"
          v-model="playerInput"
        />
      </div>
      <div id="submit-div">
        <input
          id="submit-btn"
          type="submit"
          value="submit"
          @click="checkInput"
        />
        <button @click="scrollToEnd()">Scroll to Bottom</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      currentRoom: "",
      currentRoomName: "",
      currentRoomDescription: "",
      currentRegion: "",
      currentRegionName: "",
      playerCharacter: {
        characterName: "",
        inventory: {},
      },
      playerInput: "",
      placeholder: "What will you do?",
      characterQueries: ["Man"],
      objectQueries: [
        "key",
        "wooden door",
        "keyhole",
        "wooden box",
        "metal box",
        "silver goblet",
      ],
      locationQueries: [],
      directionQueries: ["North", "West", "South", "East"],
      worldTree: {
        rooms: {
          entryPoint: {
            label: "Entry Point",
            description: `You find yourself in a room of smooth stone the color of sand.\n\nThere is a Man standing in one corner. There is a pedestal in the middle of the room with a key on it. To the North is a wooden door with a keyhole. Against one of the walls is a wooden box. Against another wall is a metal box.`,
            objects: {
              cupOfAle: {
                label: "cup of ale",
                description: "It's a tin cup with brown liquid inside.",
                isTakeable: true,
                canUseWith: {
                  man: {
                    useWithText: `You poor the ale onto the man drenching him. He yells out.`,
                    useWithFunction(worldRooms, appData) {
                      worldRooms.entryPoint.description = `You find yourself in a room of smooth stone the color of sand.\n\nThere is a Man drenched in ale standing in one corner. There is a pedestal in the middle of the room with a key on it. To the North is a wooden door with a keyhole. Against one of the walls is a wooden box. Against another wall is a metal box.`;
                      appData.currentRoomDescription =
                        worldRooms.entryPoint.description;
                      appData.characterQueries = ["drenched Man"];
                    },
                  },
                },
              },
              paper: {
                label: "paper",
                description: "A piece of paper.",
                isTakeable: true,
                canUseWith: {
                  pencil: {
                    useWithText: `You made a drawing!`,
                    useWithFunction(worldRooms, appData) {
                      delete appData.playerCharacter.inventory.paper;
                      appData.playerCharacter.inventory.drawing = {
                        label: "drawing",
                        description: "A beautiful drawing",
                      };
                    },
                  },
                },
              },
              pencil: {
                label: "pencil",
                description: "A rudimentary pencil.",
                isTakeable: true,
                canUseWith: {
                  paper: {
                    useWithText: `You're pencil has been used up.`,
                  },
                },
              },
              key: {
                label: "key",
                description: "A small bronze key",
                isTakeable: true,
                examinationFunction() {
                  this.description = "A small gold key";
                },
                canUseWith: {
                  keyhole: {
                    useWithText: `You feel a click. The door is now unlocked.`,
                    useWithFunction(worldRooms) {
                      worldRooms.entryPoint.directions.north.directionalAccess = true;
                    },
                  },
                },
              },
              woodenDoor: {
                label: "wooden door",
                description: "A tall wooden door. It looks quite sturdy.",
                isTakeable: false,
              },
              keyhole: {
                label: "keyhole",
                description: "A rusted keyhole in the door.",
                canUseWith: {
                  key: {
                    label: "key",
                  },
                },
              },
              woodenBox: {
                label: "wooden box",
                description: "A box made of wood.",
                openText: "Once open you see a silver goblet inside.",
                openFunction() {
                  delete this.openText;
                  this.description =
                    "An open wooden box with a silver goblet inside.";
                },
                pullText:
                  "You move the wooden box and find an old bloodstain on the stone floor.",
                pullFunction(worldRooms, appData) {
                  worldRooms.entryPoint.description +=
                    " There is a bloodstain on the floor where the wooden box originally was.";
                  appData.currentRoomDescription =
                    worldRooms.entryPoint.description;
                },
              },
              metalBox: {
                label: "metal box",
                description: "a box made of metal",
                pullText: "You can't pull it, it's too heavy",
              },
              silverGoblet: {
                label: "silver goblet",
                description: "A goblet made of silver.",
                isTakeable: true,
                takeText: "You now own a shiny, silver goblet",
                takeFunction(worldRooms) {
                  worldRooms.entryPoint.objects.woodenBox.description =
                    "An empty wooden box.";
                },
              },
            },
            characters: {
              man: {
                label: "Man",
                description: "A simple man, standing in the corner.",
                dialogue: [
                  `"Hello, what's your name?"`,
                  `"Oh, hi again"`,
                  `"Please stop talking to me now"`,
                ],
                talkFunction(worldRooms, appData) {
                  appData.placeholder = "What is your name?";
                  appData.checkInput = function() {
                    appData.playerCharacter.name = appData.playerInput;
                    appData.messages.push(
                      `"Oh, hello ${appData.playerCharacter.name}"`
                    );
                    appData.playerInput = "";
                    appData.placeholder = "What will you do?";
                    appData.checkInput = function() {
                      appData.determineAction();
                      appData.playerInput = "";
                    };
                  };
                },
                pushText: "Ouch! That's not nice.",
                pushFunction() {
                  this.description =
                    "A man on the floor from being unkindly pushed.";
                },
                removedDialogue: [],
                isTakeable: false,
                inventory: {},
                receivables: {
                  key: {
                    willReceive: false,
                    wontTakeText: `"You need that more than I do."`,
                  },
                  silverGoblet: {
                    willReceive: true,
                    givenText: `"Oh my, I thought I had lost this forever. Thank you ever so much."`,
                    givenFunction(worldRooms) {
                      worldRooms.entryPoint.characters.man.dialogue = [
                        `"Again I'm so grateful for you returning this goblet to me. It's a family heirloom after all."`,
                      ];
                    },
                  },
                },
                canUseWith: {
                  cupOfAle: {
                    label: "cup of ale",
                  },
                  woman: {
                    useWithText: `The man and woman kiss!`,
                  },
                },
              },
              woman: {
                label: "Woman",
                description: "A woman",
                canUseWith: {
                  man: {
                    useWithFunction(worldRooms, appData) {
                      worldRooms.entryPoint.description = `You find yourself in a room of smooth stone the color of sand.\n\nThere is a pedestal in the middle of the room with a key on it. To the North is a wooden door with a keyhole. Against one of the walls is a wooden box. Against another wall is a metal box. There are a man and woman kissing.`;
                      appData.currentRoomDescription =
                        worldRooms.entryPoint.description;
                    },
                  },
                },
              },
            },
            directions: {
              north: {
                directionalAccess: false,
                room: "visionalHall",
              },
            },
          },
          visionalHall: {
            label: "Visional Hall",
            description:
              "A long hallway will glass windows that look over a vast city.",
            directions: {
              south: {
                directionalAccess: true,
                room: "entryPoint",
                walkFunction(worldRooms) {
                  // Action-Initated Functions must be passed this.worldTree.rooms as an argument to interact with objects outside of the current one.
                  worldRooms.entryPoint.directions.north.directionalAccess = false;
                },
              },
            },
          },
        },
      },
      messages: [],
      gameStarted: false,
    };
  },
  watch: {
    messages: {
      handler() {
        if (
          this.messages[this.messages.length - 1][
            this.messages[this.messages.length - 1].length - 1
          ] != "/"
        ) {
          this.messages[this.messages.length - 1] += "\n//";
        }
        if (this.messages.length > 50) {
          this.messages.shift();
        }
        // this.scrollToEnd();
      },
      deep: true,
    },
  },
  computed: {},
  methods: {
    // Remove Begin button, sets current location, and changes divs
    startGame() {
      this.gameInfo = document.querySelector("#game-info");
      this.gameStarted = true;
      this.setRoomLocation(this.worldTree.rooms.entryPoint);
    },
    // Generic function to remove child elements
    removeChildren(parent) {
      while (parent.firstChild) {
        parent.removeChild(parent.firstChild);
      }
    },
    // Changes room location and changes room description
    setRoomLocation(room) {
      // Used in startGame and enterRoom
      console.log(room);
      this.currentRoom = room;
      this.currentRoomName = this.currentRoom.label;
      this.currentRoomDescription = this.currentRoom.description;
    },
    // Changes string to camelCase
    makeCamelCase(str) {
      return str
        .replace(/(?:^\w|[A-Z]|\b\w)/g, function(word, index) {
          return index == 0 ? word.toLowerCase() : word.toUpperCase();
        })
        .replace(/\s+/g, "");
    },

    // Highlights Text
    highlightText(text) {
      let replacedText = text;
      for (let i = 0; i < this.characterQueries.length; i++) {
        replacedText = replacedText.replace(
          new RegExp(this.characterQueries[i]),
          (match) => {
            return '<span class="character-text">' + match + `</span>`;
          }
        );
      }
      for (let i = 0; i < this.objectQueries.length; i++) {
        replacedText = replacedText.replace(
          new RegExp(this.objectQueries[i]),
          (match) => {
            return '<span class="object-text">' + match + `</span>`;
          }
        );
      }
      for (let i = 0; i < this.locationQueries.length; i++) {
        replacedText = replacedText.replace(
          new RegExp(this.locationQueries[i]),
          (match) => {
            return '<span class="location-text">' + match + `</span>`;
          }
        );
      }
      for (let i = 0; i < this.directionQueries.length; i++) {
        replacedText = replacedText.replace(
          new RegExp(this.directionQueries[i]),
          (match) => {
            return '<span class="direction-text">' + match + `</span>`;
          }
        );
      }
      return replacedText;
    },

    /////////////// Parsing Functions ///////////////

    // Gets aspect from input when there is only one
    getGeneralAspect(array) {
      let str = "";
      for (let i = 1; i < array.length; i++) {
        str = str + array[i] + " ";
      }
      let trimmedStr = str.trim();
      let camelStr = this.makeCamelCase(trimmedStr);
      return camelStr;
    },

    // Gets aspect from input when the player attempts to speak to a character
    getSpeakerAspect(array) {
      let str = "";
      for (let i = 2; i < array.length; i++) {
        str = str + array[i] + " ";
      }
      let trimmedStr = str.trim();
      let camelStr = this.makeCamelCase(trimmedStr);
      return camelStr;
    },

    // Gets aspect that the player is attempting to give to someone
    getGivenAspect(array) {
      let toIndex = array.indexOf("to");
      let str = "";
      for (let i = 1; i < toIndex; i++) {
        str = str + array[i] + " ";
      }
      let trimmedStr = str.trim();
      let camelStr = this.makeCamelCase(trimmedStr);
      return camelStr;
    },

    // Gets aspect that the player is attempting to give something to
    getReceiver(array) {
      let toIndex = array.indexOf("to");
      let str = "";
      for (let i = toIndex + 1; i < array.length; i++) {
        str = str + array[i] + " ";
      }
      let trimmedStr = str.trim();
      let camelStr = this.makeCamelCase(trimmedStr);
      return camelStr;
    },

    // Gets first of two aspects
    getFirstAspect(array) {
      let withIndex = array.indexOf("with");
      let str = "";
      for (let i = 1; i < withIndex; i++) {
        str = str + array[i] + " ";
      }
      let trimmedStr = str.trim();
      let camelStr = this.makeCamelCase(trimmedStr);
      return camelStr;
    },

    // Gets second of two aspects
    getSecondAspect(array) {
      let withIndex = array.indexOf("with");
      let str = "";
      for (let i = withIndex + 1; i < array.length; i++) {
        str = str + array[i] + " ";
      }
      let trimmedStr = str.trim();
      let camelStr = this.makeCamelCase(trimmedStr);
      return camelStr;
    },

    // Function that initiates the parser and sets the input field to blank
    checkInput() {
      this.determineAction();
      this.playerInput = "";
    },

    // Changes room to new room
    enterRoom(aspect) {
      console.log(aspect);
      let newRoom = this.currentRoom.directions[aspect].room;
      console.log(newRoom);
      console.log(this.worldTree.rooms[newRoom]);
      this.setRoomLocation(this.worldTree.rooms[newRoom]);
    },

    // Allows Player to attempt to walk in a direction
    walkDirection(aspect) {
      let originalRoom = this.currentRoom;
      console.log(this.currentRoom);
      if (
        aspect === "none" ||
        this.currentRoom.directions[aspect].directionalAccess === false
      ) {
        this.messages.push(`There's nothing in that direction.`);
      } else {
        this.enterRoom(aspect);
      }
      console.log(originalRoom);
      console.log(originalRoom.directions[aspect]);
      if (originalRoom.directions[aspect].walkFunction) {
        originalRoom.directions[aspect].walkFunction(
          this.worldTree.rooms,
          this
        );
      }
    },

    // Allows Player to examine an object or character, displaying thier description
    examineAspect(aspect) {
      if (this.currentRoom.objects[aspect]) {
        this.messages.push(this.currentRoom.objects[aspect].description);
        if (this.currentRoom.objects[aspect].examinationFunction) {
          this.currentRoom.objects[aspect].examinationFunction(
            this.worldTree.rooms,
            this
          );
        }
      } else if (this.currentRoom.characters[aspect]) {
        this.messages.push(this.currentRoom.characters[aspect].description);
        if (this.currentRoom.characters[aspect].examinationFunction) {
          this.currentRoom.characters[aspect].examinationFunction(
            this.worldTree.rooms,
            this
          );
        }
      }
    },

    // Allows Player to attempt to take an Object or Player
    takeAspect(aspect) {
      console.log(this.currentRoom.objects[aspect]);
      if (aspect in this.currentRoom.objects) {
        if (this.currentRoom.objects[aspect].isTakeable) {
          this.playerCharacter.inventory[aspect] = this.currentRoom.objects[
            aspect
          ];
          if (this.currentRoom.objects[aspect].takeText) {
            this.messages.push(this.currentRoom.objects[aspect].takeText);
          } else {
            this.messages.push(
              `You take the ${this.currentRoom.objects[aspect].label}.`
            );
          }
          if (this.currentRoom.objects[aspect].takeFunction) {
            this.currentRoom.objects[aspect].takeFunction(
              this.worldTree.rooms,
              this
            );
          }
          delete this.currentRoom.objects[aspect];
        } else {
          this.messages.push(`You can't take that with you.`);
        }
      } else if (aspect in this.currentRoom.characters) {
        if (this.currentRoom.characters[aspect].isTakeable) {
          this.playerCharacter.inventory.push(
            this.currentRoom.characters[aspect]
          );
          if (this.currentRoom.characters[aspect].takeText) {
            this.messages.push(this.currentRoom.characters[aspect].takeText);
          } else {
            this.messages.push(
              `You take the ${this.currentRoom.characters[aspect].label}.`
            );
          }
          if (this.currentRoom.characters[aspect].takeFunction) {
            this.currentRoom.characters[aspect].takeFunction(
              this.worldTree.rooms,
              this
            );
          }
          delete this.currentRoom.characters[aspect];
        } else {
          this.messages.push(`You can't take that with you.`);
        }
      }
    },

    openAspect(aspect) {
      if (aspect in this.currentRoom.objects) {
        if (!("openText" in this.currentRoom.objects[aspect])) {
          this.messages.push("You can't open that.");
          return;
        }
        this.messages.push(this.currentRoom.objects[aspect].openText);
        if (this.currentRoom.objects[aspect].openFunction) {
          this.currentRoom.objects[aspect].openFunction(
            this.worldTree.rooms,
            this
          );
        }
      } else if (aspect in this.currentRoom.characters) {
        if (!("openText" in this.currentRoom.characters[aspect])) {
          this.messages.push("You can't open that.");
          return;
        }
        this.messages.push(this.currentRoom.characters[aspect].openText);
        if (this.currentRoom.characters[aspect].openFunction) {
          this.currentRoom.characters[aspect].openFunction(
            this.worldTree.rooms,
            this
          );
        }
      }
    },

    pushAspect(aspect) {
      if (aspect in this.currentRoom.objects) {
        if (!("pushText" in this.currentRoom.objects[aspect])) {
          this.messages.push("You can't push that.");
          return;
        }
        this.messages.push(this.currentRoom.objects[aspect].pushText);
        if (this.currentRoom.objects[aspect].pushFunction) {
          this.currentRoom.objects[aspect].pushFunction(
            this.worldTree.rooms,
            this
          );
        }
      } else if (aspect in this.currentRoom.characters) {
        if (!("pushText" in this.currentRoom.characters[aspect])) {
          this.messages.push("You can't push them.");
          return;
        }
        this.messages.push(this.currentRoom.characters[aspect].pushText);
        if (this.currentRoom.characters[aspect].pushFunction) {
          this.currentRoom.characters[aspect].pushFunction(
            this.worldTree.rooms,
            this
          );
        }
      }
    },

    pullAspect(aspect) {
      if (aspect in this.currentRoom.objects) {
        if (!("pullText" in this.currentRoom.objects[aspect])) {
          this.messages.push("You can't pull that.");
          return;
        }
        this.messages.push(this.currentRoom.objects[aspect].pullText);
        if (this.currentRoom.objects[aspect].pullFunction) {
          this.currentRoom.objects[aspect].pullFunction(
            this.worldTree.rooms,
            this
          );
        }
      } else if (aspect in this.currentRoom.characters) {
        if (!("pushText" in this.currentRoom.characters[aspect])) {
          this.messages.push("You can't push them.");
          return;
        }
        this.messages.push(this.currentRoom.characters[aspect].pullText);
        if (this.currentRoom.characters[aspect].pullFunction) {
          this.currentRoom.characters[aspect].pullFunction(
            this.worldTree.rooms,
            this
          );
        }
      }
    },

    talkToAspect(aspect) {
      if (aspect in this.currentRoom.objects) {
        if (!("dialogue" in this.currentRoom.objects[aspect])) {
          this.messages.push("It doesn't have anything to say.");
          return;
        }
        if (this.currentRoom.objects[aspect].dialogue.length > 1) {
          this.messages.push(this.currentRoom.objects[aspect].dialogue[0]);
          this.currentRoom.objects[aspect].removedDialogue.push(
            this.currentRoom.objects[aspect].dialogue[0]
          );
          this.currentRoom.objects[aspect].dialogue.shift();
        } else {
          this.messages.push(this.currentRoom.objects[aspect].dialogue[0]);
        }
        if (this.currentRoom.objects[aspect].talkFunction) {
          this.currentRoom.objects[aspect].talkFunction(
            this.worldTree.rooms,
            this
          );
        }
      } else if (aspect in this.currentRoom.characters) {
        if (!("dialogue" in this.currentRoom.characters[aspect])) {
          this.messages.push("They don't have anything to say.");
          return;
        }
        if (this.currentRoom.characters[aspect].dialogue.length > 1) {
          this.messages.push(this.currentRoom.characters[aspect].dialogue[0]);
          this.currentRoom.characters[aspect].removedDialogue.push(
            this.currentRoom.characters[aspect].dialogue[0]
          );
          this.currentRoom.characters[aspect].dialogue.shift();
        } else {
          this.messages.push(this.currentRoom.characters[aspect].dialogue[0]);
        }
        if (this.currentRoom.characters[aspect].talkFunction) {
          this.currentRoom.characters[aspect].talkFunction(
            this.worldTree.rooms,
            this
          );
        }
      }
    },

    giveAspect(aspect, receiver) {
      console.log(aspect, receiver);
      if (!this.playerCharacter.inventory[aspect]) {
        this.messages.push(`You don't have ${aspect} with you.`);
      } else {
        if (!this.currentRoom.characters[receiver].receivables[aspect]) {
          this.messages.push(`"I don't want that."`);
        } else if (
          !this.currentRoom.characters[receiver].receivables[aspect].willReceive
        ) {
          if (
            !(
              "wontTakeText" in
              this.currentRoom.characters[receiver].receivables[aspect]
            )
          ) {
            this.messages.push(`"I don't want that."`);
          } else {
            this.messages.push(
              this.currentRoom.characters[receiver].receivables[aspect]
                .wontTakeText
            );
          }
        } else {
          this.currentRoom.characters[receiver].inventory[
            aspect
          ] = this.playerCharacter.inventory[aspect];

          this.messages.push(
            this.currentRoom.characters[receiver].receivables[aspect].givenText
          );
          console.log(this.currentRoom.characters[receiver].inventory);
          delete this.playerCharacter.inventory[aspect];
          console.log(this.playerCharacter.inventory);
          if (
            this.currentRoom.characters[receiver].receivables[aspect]
              .givenFunction
          ) {
            this.currentRoom.characters[receiver].receivables[
              aspect
            ].givenFunction(this.worldTree.rooms, this);
          }
        }
      }
    },

    useAspect(firstAspect, secondAspect) {
      if (
        // If the first object or character is not in the player's inventory or in the room
        !(firstAspect in this.playerCharacter.inventory) &&
        !(firstAspect in this.currentRoom.objects) &&
        !(firstAspect in this.currentRoom.characters)
      ) {
        this.messages.push(`You don't have ${firstAspect} with you.`);
      } else if (
        // If the second object or character is not in the player's inventory or in the room
        !(secondAspect in this.playerCharacter.inventory) &&
        !(secondAspect in this.currentRoom.objects) &&
        !(secondAspect in this.currentRoom.characters)
      ) {
        this.messages.push(`You don't have access to ${secondAspect}.`);
      } else if (
        // If both objects or characters are in the player's inventory
        firstAspect in this.playerCharacter.inventory &&
        secondAspect in this.playerCharacter.inventory &&
        secondAspect in
          this.playerCharacter.inventory[firstAspect].canUseWith &&
        firstAspect in this.playerCharacter.inventory[secondAspect].canUseWith
      ) {
        if (
          // If there is text for using the objects attached to the first object, push it to messages
          this.playerCharacter.inventory[firstAspect].canUseWith[secondAspect]
            .useWithText
        ) {
          this.messages.push(
            this.playerCharacter.inventory[firstAspect].canUseWith[secondAspect]
              .useWithText
          );
        }
        if (
          // If there is text for using the objects attached to the second object, push it to messages
          this.playerCharacter.inventory[secondAspect].canUseWith[firstAspect]
            .useWithText
        ) {
          this.messages.push(
            this.playerCharacter.inventory[secondAspect].canUseWith[firstAspect]
              .useWithText
          );
        }
        if (
          // If the first object or character has a function, trigger it
          this.playerCharacter.inventory[firstAspect].canUseWith[secondAspect]
            .useWithFunction
        ) {
          this.playerCharacter.inventory[firstAspect].canUseWith[
            secondAspect
          ].useWithFunction(this.worldTree.rooms, this);
        }
        if (
          // If the second object or character has a function, trigger it
          this.playerCharacter.inventory[secondAspect].canUseWith[firstAspect]
            .useWithFunction
        ) {
          this.playerCharacter.inventory[secondAspect].canUseWith[
            firstAspect
          ].useWithFunction(this.worldTree.rooms);
        }
      } else if (
        // If the first object or character is in the player's inventory and the second object is in the room
        firstAspect in this.playerCharacter.inventory &&
        secondAspect in this.currentRoom.objects &&
        secondAspect in
          this.playerCharacter.inventory[firstAspect].canUseWith &&
        firstAspect in this.currentRoom.objects[secondAspect].canUseWith
      ) {
        if (
          // If there is text for using the objects attached to the first object, push it to messages
          this.playerCharacter.inventory[firstAspect].canUseWith[secondAspect]
            .useWithText
        ) {
          this.messages.push(
            this.playerCharacter.inventory[firstAspect].canUseWith[secondAspect]
              .useWithText
          );
        }
        if (
          // If there is text for using the objects attached to the second object, push it to messages
          this.currentRoom.objects[secondAspect].canUseWith[firstAspect]
            .useWithText
        ) {
          this.messages.push(
            this.currentRoom.objects[secondAspect].canUseWith[firstAspect]
              .useWithText
          );
        }
        if (
          // If the first object or character has a function, trigger it
          this.playerCharacter.inventory[firstAspect].canUseWith[secondAspect]
            .useWithFunction
        ) {
          this.playerCharacter.inventory[firstAspect].canUseWith[
            secondAspect
          ].useWithFunction(this.worldTree.rooms, this);
        }
        if (
          // If the second object has a function, trigger it
          this.currentRoom.objects[secondAspect].canUseWith[firstAspect]
            .useWithFunction
        ) {
          this.currentRoom.objects[secondAspect].canUseWith[
            firstAspect
          ].useWithFunction(this.worldTree.rooms);
        }
      } else if (
        // If the first object or character is in the player's inventory and the second character is in the room
        firstAspect in this.playerCharacter.inventory &&
        secondAspect in this.currentRoom.characters &&
        secondAspect in
          this.playerCharacter.inventory[firstAspect].canUseWith &&
        firstAspect in this.currentRoom.characters[secondAspect].canUseWith
      ) {
        if (
          // If there is text for using the objects attached to the first object, push it to messages
          this.playerCharacter.inventory[firstAspect].canUseWith[secondAspect]
            .useWithText
        ) {
          this.messages.push(
            this.playerCharacter.inventory[firstAspect].canUseWith[secondAspect]
              .useWithText
          );
        }
        if (
          // If there is text for using the objects attached to the second character, push it to messages
          this.currentRoom.characters[secondAspect].canUseWith[firstAspect]
            .useWithText
        ) {
          this.messages.push(
            this.currentRoom.characters[secondAspect].canUseWith[firstAspect]
              .useWithText
          );
        }
        if (
          // If the first object or character has a function, trigger it
          this.playerCharacter.inventory[firstAspect].canUseWith[secondAspect]
            .useWithFunction
        ) {
          this.playerCharacter.inventory[firstAspect].canUseWith[
            secondAspect
          ].useWithFunction(this.worldTree.rooms, this);
        }
        if (
          // If the second object has a function, trigger it
          this.currentRoom.characters[secondAspect].canUseWith[firstAspect]
            .useWithFunction
        ) {
          this.currentRoom.characters[secondAspect].canUseWith[
            firstAspect
          ].useWithFunction(this.worldTree.rooms);
        }
      } else if (
        // If the first object is in the room and the second object or character is in the player's inventory
        firstAspect in this.currentRoom.objects &&
        secondAspect in this.playerCharacter.inventory &&
        secondAspect in this.currentRoom.objects[firstAspect].canUseWith &&
        firstAspect in this.playerCharacter.inventory[secondAspect].canUseWith
      ) {
        if (
          // If there is text for using the objects attached to the first object, push it to messages
          this.currentRoom.objects[firstAspect].canUseWith[secondAspect]
            .useWithText
        ) {
          this.messages.push(
            this.currentRoom.objects[firstAspect].canUseWith[secondAspect]
              .useWithText
          );
        }
        if (
          // If there is text for using the objects attached to the second object, push it to messages
          this.playerCharacter.inventory[secondAspect].canUseWith[firstAspect]
            .useWithText
        ) {
          this.messages.push(
            this.playerCharacter.inventory[secondAspect].canUseWith[firstAspect]
              .useWithText
          );
        }
        if (
          // If the first object has a function, trigger it
          this.currentRoom.objects[firstAspect].canUseWith[secondAspect]
            .useWithFunction
        ) {
          this.currentRoom.objects[firstAspect].canUseWith[
            secondAspect
          ].useWithFunction(this.worldTree.rooms, this);
        }
        if (
          // If the second object or character has a function, trigger it
          this.playerCharacter.inventory[secondAspect].canUseWith[firstAspect]
            .useWithFunction
        ) {
          this.playerCharacter.inventory[secondAspect].canUseWith[
            firstAspect
          ].useWithFunction(this.worldTree.rooms);
        }
      } else if (
        // If both objects are in the room
        firstAspect in this.currentRoom.objects &&
        secondAspect in this.currentRoom.objects &&
        secondAspect in this.currentRoom.objects[firstAspect].canUseWith &&
        firstAspect in this.currentRoom.objects[secondAspect].canUseWith
      ) {
        if (
          // If there is text for using the objects attached to the first object, push it to messages
          this.currentRoom.objects[firstAspect].canUseWith[secondAspect]
            .useWithText
        ) {
          this.messages.push(
            this.currentRoom.objects[firstAspect].canUseWith[secondAspect]
              .useWithText
          );
        }
        if (
          // If there is text for using the objects attached to the second object, push it to messages
          this.currentRoom.objects[secondAspect].canUseWith[firstAspect]
            .useWithText
        ) {
          this.messages.push(
            this.currentRoom.objects[secondAspect].canUseWith[firstAspect]
              .useWithText
          );
        }
        if (
          // If the first object has a function, trigger it
          this.currentRoom.objects[firstAspect].canUseWith[secondAspect]
            .useWithFunction
        ) {
          this.currentRoom.objects[firstAspect].canUseWith[
            secondAspect
          ].useWithFunction(this.worldTree.rooms, this);
        }
        if (
          // If the second object or character has a function, trigger it
          this.currentRoom.objects[secondAspect].canUseWith[firstAspect]
            .useWithFunction
        ) {
          this.currentRoom.objects[secondAspect].canUseWith[
            firstAspect
          ].useWithFunction(this.worldTree.rooms);
        }
      } else if (
        // If the object is in the room and the character is in the room
        firstAspect in this.currentRoom.objects &&
        secondAspect in this.currentRoom.characters &&
        secondAspect in this.currentRoom.objects[firstAspect].canUseWith &&
        firstAspect in this.currentRoom.characters[secondAspect].canUseWith
      ) {
        if (
          // If there is text for using the objects attached to the first object, push it to messages
          this.currentRoom.objects[firstAspect].canUseWith[secondAspect]
            .useWithText
        ) {
          this.messages.push(
            this.currentRoom.objects[firstAspect].canUseWith[secondAspect]
              .useWithText
          );
        }
        if (
          // If there is text for using the objects attached to the second object, push it to messages
          this.currentRoom.characters[secondAspect].canUseWith[firstAspect]
            .useWithText
        ) {
          this.messages.push(
            this.currentRoom.characters[secondAspect].canUseWith[firstAspect]
              .useWithText
          );
        }
        if (
          // If the object has a function, trigger it
          this.currentRoom.objects[firstAspect].canUseWith[secondAspect]
            .useWithFunction
        ) {
          this.currentRoom.objects[firstAspect].canUseWith[
            secondAspect
          ].useWithFunction(this.worldTree.rooms, this);
        }
        if (
          // If the character has a function, trigger it
          this.currentRoom.characters[secondAspect].canUseWith[firstAspect]
            .useWithFunction
        ) {
          this.currentRoom.characters[secondAspect].canUseWith[
            firstAspect
          ].useWithFunction(this.worldTree.rooms);
        }
      } else if (
        // If the first character is in the room and the second object or character is in the player's inventory
        firstAspect in this.currentRoom.characters &&
        secondAspect in this.playerCharacter.inventory &&
        secondAspect in this.currentRoom.characters[firstAspect].canUseWith &&
        firstAspect in this.playerCharacter.inventory[secondAspect].canUseWith
      ) {
        if (
          // If there is text for using the objects attached to the first object, push it to messages
          this.currentRoom.characters[firstAspect].canUseWith[secondAspect]
            .useWithText
        ) {
          this.messages.push(
            this.currentRoom.characters[firstAspect].canUseWith[secondAspect]
              .useWithText
          );
        }
        if (
          // If there is text for using the objects attached to the second object, push it to messages
          this.playerCharacter.inventory[secondAspect].canUseWith[firstAspect]
            .useWithText
        ) {
          this.messages.push(
            this.playerCharacter.inventory[secondAspect].canUseWith[firstAspect]
              .useWithText
          );
        }
        if (
          // If the first character has a function, trigger it
          this.currentRoom.characters[firstAspect].canUseWith[secondAspect]
            .useWithFunction
        ) {
          this.currentRoom.characters[firstAspect].canUseWith[
            secondAspect
          ].useWithFunction(this.worldTree.rooms, this);
        }
        if (
          // If the second object or character has a function, trigger it
          this.playerCharacter.inventory[secondAspect].canUseWith[firstAspect]
            .useWithFunction
        ) {
          this.playerCharacter.inventory[secondAspect].canUseWith[
            firstAspect
          ].useWithFunction(this.worldTree.rooms, this);
        }
      } else if (
        // If the character is in the room and the object is in the room
        firstAspect in this.currentRoom.characters &&
        secondAspect in this.currentRoom.objects &&
        secondAspect in this.currentRoom.characters[firstAspect].canUseWith &&
        firstAspect in this.currentRoom.objects[secondAspect].canUseWith
      ) {
        if (
          // If there is text for using the objects attached to the first object, push it to messages
          this.currentRoom.characters[firstAspect].canUseWith[secondAspect]
            .useWithText
        ) {
          this.messages.push(
            this.currentRoom.characters[firstAspect].canUseWith[secondAspect]
              .useWithText
          );
        }
        if (
          // If there is text for using the objects attached to the second object, push it to messages
          this.currentRoom.objects[secondAspect].canUseWith[firstAspect]
            .useWithText
        ) {
          this.messages.push(
            this.currentRoom.objects[secondAspect].canUseWith[firstAspect]
              .useWithText
          );
        }
        if (
          // If the first character has a function, trigger it
          this.currentRoom.characters[firstAspect].canUseWith[secondAspect]
            .useWithFunction
        ) {
          this.currentRoom.characters[firstAspect].canUseWith[
            secondAspect
          ].useWithFunction(this.worldTree.rooms, this);
        }
        if (
          // If the second object or character has a function, trigger it
          this.currentRoom.objects[secondAspect].canUseWith[firstAspect]
            .useWithFunction
        ) {
          this.currentRoom.objects[secondAspect].canUseWith[
            firstAspect
          ].useWithFunction(this.worldTree.rooms, this);
        }
      } else if (
        // If both characters are in the room
        firstAspect in this.currentRoom.characters &&
        secondAspect in this.currentRoom.characters &&
        secondAspect in this.currentRoom.characters[firstAspect].canUseWith &&
        firstAspect in this.currentRoom.characters[secondAspect].canUseWith
      ) {
        if (
          // If there is text for using the characters attached to the first character, push it to messages
          this.currentRoom.characters[firstAspect].canUseWith[secondAspect]
            .useWithText
        ) {
          this.messages.push(
            this.currentRoom.characters[firstAspect].canUseWith[secondAspect]
              .useWithText
          );
        }
        if (
          // If there is text for using the characters attached to the second character, push it to messages
          this.currentRoom.characters[secondAspect].canUseWith[firstAspect]
            .useWithText
        ) {
          this.messages.push(
            this.currentRoom.characters[secondAspect].canUseWith[firstAspect]
              .useWithText
          );
        }
        if (
          // If the first character has a function, trigger it
          this.currentRoom.characters[firstAspect].canUseWith[secondAspect]
            .useWithFunction
        ) {
          this.currentRoom.characters[firstAspect].canUseWith[
            secondAspect
          ].useWithFunction(this.worldTree.rooms, this);
        }
        if (
          // If the second object or character has a function, trigger it
          this.currentRoom.characters[secondAspect].canUseWith[firstAspect]
            .useWithFunction
        ) {
          this.currentRoom.characters[secondAspect].canUseWith[
            firstAspect
          ].useWithFunction(this.worldTree.rooms, this);
        }
      } else {
        console.log("Yes");
        this.messages.push(`You can't use those with each other.`);
      }
    },

    checkInventory() {
      let inventoryList = `You are holding:`;
      for (const item in this.playerCharacter.inventory) {
        inventoryList += `\n- ${this.playerCharacter.inventory[item].label}`;
      }
      this.messages.push(inventoryList);
    },

    determineAction() {
      let inputArray = this.playerInput.toLowerCase().split(" ");
      let aspect = "";
      if (inputArray[0] === "walk" || inputArray[0] === "go") {
        aspect = inputArray[1];
        this.walkDirection(aspect);
      } else if (inputArray[0] === "examine") {
        aspect = this.getGeneralAspect(inputArray);
        this.examineAspect(aspect);
      } else if (inputArray[0] === "take") {
        aspect = this.getGeneralAspect(inputArray);
        this.takeAspect(aspect);
      } else if (inputArray[0] === "open") {
        aspect = this.getGeneralAspect(inputArray);
        this.openAspect(aspect);
      } else if (inputArray[0] === "push") {
        aspect = this.getGeneralAspect(inputArray);
        this.pushAspect(aspect);
      } else if (inputArray[0] === "pull") {
        aspect = this.getGeneralAspect(inputArray);
        this.pullAspect(aspect);
      } else if (inputArray[0] === "talk" && inputArray[1] === "to") {
        aspect = this.getSpeakerAspect(inputArray);
        this.talkToAspect(aspect);
      } else if (inputArray[0] === "give") {
        let aspect = this.getGivenAspect(inputArray);
        let receiver = this.getReceiver(inputArray);
        this.giveAspect(aspect, receiver);
      } else if (inputArray[0] === "use") {
        let firstAspect = this.getFirstAspect(inputArray);
        let secondAspect = this.getSecondAspect(inputArray);
        this.useAspect(firstAspect, secondAspect);
      } else if (inputArray[0] === "?") {
        if (inputArray.length > 1) {
          aspect = this.getGeneralAspect(inputArray);
          this.getHelpWith(aspect);
        } else {
          this.getHelp();
        }
      } else if (inputArray[0] === "check" && inputArray[1] === "inventory") {
        this.checkInventory();
      }
    },

    scrollToEnd() {
      let actionResponseDiv = this.$refs.actionResponseDiv;
      actionResponseDiv.scrollTop = actionResponseDiv.scrollHeight;
    },
  },
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,200;0,400;0,900;1,200;1,400;1,900&display=swap");
@import url("https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap");
@import url("https://fonts.googleapis.com/css2?family=Fira+Code:wght@400;500;600;700&display=swap");

body {
  background-color: rgb(41, 41, 41);
}

#header-div {
  display: block;
  margin-left: auto;
  margin-right: auto;
  margin-top: 0;
  margin-bottom: 0;
  padding: 0;
  width: 75%;
}

h1 {
  font-family: monospace;
  font-weight: 300;
  font-style: italic;
  font-size: 5em;
  text-align: center;
  color: whitesmoke;
  margin: 0.25em auto 0.2em;
}

.header-span {
  color: rgb(255, 227, 68);
}

#subtitle {
  font-family: "Fira Code", monospace;
  font-weight: 300;
  font-style: normal;
  font-size: 1em;
  text-align: center;
  color: whitesmoke;
  margin: 0 auto 2em;
}

#console {
  background-color: grey;
  margin: 1em;
  height: 40em;
}

#game-info {
  position: relative;
  top: 1.75em;
  background-color: rgb(66, 66, 66);
  height: 30em;
  margin: 0 2em;
}

#location-div {
  display: flex;
  position: relative;
  justify-content: space-between;
  margin: 0 1em;
}

#RegionLocation {
  display: block;
  margin: 0;
}

#roomLocation {
  display: block;
  margin: 0;
}

.location-header {
  font-family: "Fira Code", monospace;
  font-size: 1.2em;
  color: hsl(82, 40.8%, 60%);
  margin: 0;
}

#welcome-div {
  display: block;
  position: relative;
  height: 95%;
  margin: 0 2em;
  padding: 0 2em 0 0;
  overflow-y: auto;
}

#text-div {
  display: block;
  position: relative;
  background-color: rgb(55, 55, 55);
  margin: 0 auto;
  width: 83em;
  height: 11em;
  overflow-y: auto;
}

#description-div {
  display: block;
  position: relative;
  background-color: rgb(55, 55, 55);
  margin: 0 1em 1em;
  padding: 0 1em;
  height: 12em;
  overflow-y: auto;
}

#action-response-div {
  display: block;
  position: relative;
  background-color: rgb(55, 55, 55);
  margin: 0 1em 2em;
  padding: 1em 1em 0;
  height: 12em;
  overflow-y: auto;
}

#welcome-message {
  font-family: "Fira Code", monospace;
  font-size: 1.5em;
  color: rgb(255, 227, 68);
}

.yggdrasil {
  font-family: "Fira Code", monospace;
  color: rgb(255, 227, 68);
}

.game-text {
  font-family: "Montserrat";
  font-weight: 500;
  font-size: 1em;
  color: whitesmoke;
  line-height: 1.6;
  white-space: pre-line;
  text-align: justify;
  /* margin: 0; */
}

.location-header-text {
  font-family: "Fira Code", monospace;
  color: hsl(82, 40.8%, 60%);
}

.character-text {
  transition: color 1.2s ease;
  /* font-family: "Fira Code", monospace; */
}

.character-text:hover {
  /* font-family: "Fira Code", monospace; */
  color: hsl(0, 40.8%, 60%);
}

.object-text {
  transition: color 1.2s ease;
  /* font-family: "Fira Code", monospace; */
}

.object-text:hover {
  /* font-family: "Fira Code", monospace; */
  color: hsl(248, 40.8%, 60%);
}

.location-text {
  transition: color 1.2s ease;
  /* font-family: "Fira Code", monospace; */
}

.location-text:hover {
  /* font-family: "Fira Code", monospace; */
  color: hsl(82, 40.8%, 60%);
}

.direction-text {
  transition: color 1.2s ease;
  /* font-family: "Fira Code", monospace; */
}

.direction-text:hover {
  /* font-family: "Fira Code", monospace; */
  color: hsl(39, 40.8%, 60%);
}

.verb-text {
  font-family: "Fira Code", monospace;
  color: hsl(43, 74.4%, 49%);
}

#player-interface {
  display: flex;
  flex-direction: row;
  position: relative;
  top: 3em;
  background-color: rgb(66, 66, 66);
  height: 5em;
  margin: 0 2em;
}

#begin-div {
  display: flex;
  align-items: center;
  margin: 0 2em;
}

#begin-btn {
  font-family: "Fira Code", monospace;
  font-size: 1.75em;
  width: 5em;
  height: 1.6em;
  background-color: hsl(43, 74.4%, 49%);
  color: whitesmoke;
  border-style: none;
}

#label-div {
  display: flex;
  align-items: center;
  margin: 0 2em;
}

#input-div {
  display: flex;
  align-items: center;
}

#player-input {
  font-family: "Fira Code", monospace;
  padding: 0 1em;
  margin: 0 0 0 1rem;
  color: whitesmoke;
  font-size: 1em;
  height: 3em;
  border-style: none;
  background-color: rgb(41, 41, 41);
}

#player-input:focus {
  outline: none;
}

#submit-button:focus {
  outline: none;
}

#submit-div {
  display: flex;
  align-items: center;
  margin: 0 2em;
}

#submit-btn {
  background-color: goldenrod;
  color: whitesmoke;
  font-family: Montserrat;
  font-size: 1em;
  width: 5em;
  height: 3em;
  border-style: none;
}
</style>
