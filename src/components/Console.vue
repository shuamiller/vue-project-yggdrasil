<template>
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
          <p id="Region-text" class="location-text">{{ currentRegionName }}</p>
        </div>
        <div id="room-location">
          <p id="room-text" class="location-text">{{ currentRoomName }}</p>
        </div>
      </div>
      <div v-if="gameStarted" id="description-div">
        <p id="room-description-text" class="game-text">
          {{ currentRoomDescription }}
        </p>
      </div>
      <div v-if="gameStarted" id="action-response-div">
        <p v-for="(message, index) in messages" :key="index" class="game-text">
          {{ message }}
        </p>
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
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Console",
  props: {
    msg: String,
  },
  data() {
    return {
      currentRoom: "",
      currentRoomName: "",
      currentRoomDescription: "",
      currentRegion: "",
      currentRegionName: "",
      playerCharacter: {
        name: "",
        inventory: [],
      },
      playerInput: "",
      placeholder: "What will you do?",
      worldTree: {
        rooms: {
          entryPoint: {
            name: "Entry Points",
            description: `You find yourself in a room of smooth stone the color of sand. You don't remember how you came to be here. You don't remember who you are. \n\n This room seems to be illuminated without a source. Ahead of you, there is a recessed wall that could be a doorway, but there is no opening. To the left of the recess, a circular portal of dark glass is set in the wall.`,
            objects: {
              doorway: {
                description: `A smooth surface, the same color and texture as the rest of the room but recessed into the North wall.`,
                openText: `It doesn't appear to open in any way.`,
                pushText: `You push against the flat surface, but it doesn't give.`,
                pullText: `There isn't any way to pull it.`,
                dialogue: [`It says nothing.`, `It still says nothing...`],
                removedDialogue: [],
                isTakeable: false,
              },
              darkGlass: {
                name: `dark glass`,
                description: `A circular portal of dark glass set into the wall. It is about the size of your head. As you look at it, the glass like surface begins to ripple suddenly. You hear a voice come from the portal. As the voice speaks, the ripple grows.\n\n"Oh, another Convert. It's been some time. I didn't know there would be more of you, but I know not what happens beyond the deep. What is your name, Convert?"`,
                isTakeable: false,
                examinationFunction() {
                  this.checkInput = function () {
                    this.playerCharacter.name = this.playerInput;
                    this.messages.push(
                      `"It's good to meet you, ${this.playerCharacter.name}! I'm Greeter,Dif. Of course you'll have heard of me from before your Conver- Oh! You appear to be missing your PED. How unusual...\n\nSomething's wrong. I'm sorry about the inconvenince. NOrmally, I'd send you to the Council, but as they've dispersed, I'll have to take care of this somehow. Meet me in the Hall,Annals across the city; We can try to figure this out there.\n\nI'm sorry the circumstances are so strange, but nonetheless... Welcome to Yggdrasil."\n\nThe doorway before you to the North splits down the middle, and the two halves recede into the walls creating an opening.`
                    );
                  };
                },
              },
            },
            characters: {},
            directions: {
              north: {
                directionalAccess: false,
                room: "VisionalHall",
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
      handler(val) {
        console.log(val);
        if (this.messages.length > 50) {
          this.messages.shift();
        }
        let actionResponseDiv = document.querySelector("#action-response-div");
        actionResponseDiv.scrollTop = actionResponseDiv.scrollHeight;
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
      this.currentRoom = room;
      this.currentRoomName = room.name;
      this.currentRoomDescription = this.currentRoom.description;
    },
    // Changes string to camelCase
    makeCamelCase(str) {
      return str
        .replace(/(?:^\w|[A-Z]|\b\w)/g, function (word, index) {
          return index == 0 ? word.toLowerCase() : word.toUpperCase();
        })
        .replace(/\s+/g, "");
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
      console.log(this.messages.length);
    },
    // Changes room to new room
    enterRoom(aspect) {
      let newRoom = this.currentRoom.directions[aspect].room;
      this.setRoomLocation(this.worldTree.rooms[newRoom]);
    },
    // Allows Player to attempt to walk in a direction
    walkDirection(aspect) {
      if (
        aspect === "none" ||
        this.currentRoom.directions[aspect].directionalAccess === false
      ) {
        this.messages.push(`There's nothing in that direction.`);
      } else {
        this.enterRoom(aspect);
      }
      if (this.currentRoom.walkFunction) {
        this.currentRoom.walkFunction();
      }
    },
    // Allows Player to examine an object or character, displaying thier description
    examineAspect(aspect) {
      if (this.currentRoom.objects[aspect]) {
        this.messages.push(this.currentRoom.objects[aspect].description);
        if (this.currentRoom.objects[aspect].examinationFunction) {
          this.currentRoom.objects[aspect].examinationFunction();
        }
      } else if (this.currentRoom.characters[aspect]) {
        this.messages.push(this.currentRoom.characters[aspect].description);
        if (this.currentRoom.characters[aspect].examinationFunction) {
          this.currentRoom.characters[aspect].examinationFunction();
        }
      }
    },
    // Allows Player to attempt to take an Object or Player
    takeAspect(aspect) {
      if (aspect in this.currentRoom.objects) {
        if (this.currentRoom.objects[aspect].isTakeable) {
          this.playerCharacter.inventory.push(this.currentRoom.objects[aspect]);
          delete this.currentRoom.objects[aspect];
          if (this.currentRoom.objects[aspect].takeText) {
            this.messages.push(this.currentRoom.objects[aspect].takeText);
          } else {
            this.messages.push(
              `You take the ${this.currentRoom.objects[aspect].name}.`
            );
          }
          if (this.currentRoom.objects[aspect].takeFunction) {
            this.currentRoom.objects[aspect].takeFunction();
          }
        } else {
          this.messages.push(``);
        }
      } else if (aspect in this.currentRoom.characters) {
        if (this.currentRoom.characters[aspect].isTakeable) {
          this.playerCharacter.inventory.push(
            this.currentRoom.characters[aspect]
          );
          delete this.currentRoom.characters[aspect];
          if (this.currentRoom.characters[aspect].takeText) {
            this.messages.push(this.currentRoom.characters[aspect].takeText);
          } else {
            this.messages.push(
              `You take the ${this.currentRoom.characters[aspect].name}.`
            );
          }
          if (this.currentRoom.characters[aspect].takeFunction) {
            this.currentRoom.characters[aspect].takeFunction();
          }
        } else {
          this.messages.push(`You can't take that with you.`);
        }
      }
    },
    openAspect(aspect) {
      if (aspect in this.currentRoom.objects) {
        this.messages.push(this.currentRoom.objects[aspect].openText);
        if (this.currentRoom.objects[aspect].openFunction) {
          this.currentRoom.objects[aspect].openFunction();
        }
      } else if (aspect in this.currentRoom.characters) {
        this.messages.push(this.currentRoom.characters[aspect].openText);
        if (this.currentRoom.characters[aspect].openFunction) {
          this.currentRoom.characters[aspect].openFunction();
        }
      }
    },
    pushAspect(aspect) {
      if (aspect in this.currentRoom.objects) {
        this.messages.push(this.currentRoom.objects[aspect].pushText);
        if (this.currentRoom.objects[aspect].pushFunction) {
          this.currentRoom.objects[aspect].pushFunction();
        }
      } else if (aspect in this.currentRoom.characters) {
        this.messages.push(this.currentRoom.characters[aspect].pushText);
        if (this.currentRoom.characters[aspect].pushFunction) {
          this.currentRoom.characters[aspect].pushFunction();
        }
      }
    },
    pullAspect(aspect) {
      if (aspect in this.currentRoom.objects) {
        this.messages.push(this.currentRoom.objects[aspect].pullText);
        if (this.currentRoom.objects[aspect].pullFunction) {
          this.currentRoom.objects[aspect].pullFunction();
        }
      } else if (aspect in this.currentRoom.characters) {
        this.messages.push(this.currentRoom.characters[aspect].pullText);
        if (this.currentRoom.characters[aspect].pullFunction) {
          this.currentRoom.characters[aspect].pullFunction();
        }
      }
    },
    talkToAspect(aspect) {
      if (aspect in this.currentRoom.objects) {
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
          this.currentRoom.objects[aspect].talkFunction();
        }
      } else if (aspect in this.currentRoom.characters) {
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
          this.currentRoom.characters[aspect].talkFunction();
        }
      }
    },
    giveFunction(aspect, receiver) {
      if (!this.playerCharacter.inventory[aspect]) {
        this.messages.push(`You don't have ${aspect} with you.`);
      } else {
        if (!this.currentRoom[receiver].receivable[aspect]) {
          this.messages.push(this.currentRoom[receiver].wontTakeText);
        } else {
          this.currentRoom[receiver].inventory.push(
            this.playerCharacter.inventory[aspect]
          );
          const aspectIndex = this.playerCharacter.inventory.indexOf(
            this.playerCharacter.inventory[aspect]
          );
          this.playerCharacter.inventory.splice(aspectIndex, 1);
        }
      }
    },
    useAspect(firstAspect, secondAspect) {
      // Refactor code from script.js
      firstAspect;
      secondAspect;
    },
    checkInventory() {
      let inventoryList = `You are holding:`;
      for (const item in this.playerCharacter.inventory) {
        inventoryList += `\n*${item}`;
      }
      this.messages.push(inventoryList);
    },
    determineAction() {
      let inputArray = this.playerInput.toLowerCase().split(" ");
      let aspect = "";
      if (inputArray[0] === "walk" || inputArray[0] === "go") {
        aspect = inputArray[1];
        console.log(aspect);
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
        let aspect1 = this.getAspect1(inputArray);
        let aspect2 = this.getAspect2(inputArray);
        this.useAspect(aspect1, aspect2);
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
      let actionResponseDiv = this.$el.querySelector("#action-response-div");
      actionResponseDiv.scrollTop = actionResponseDiv.scrollHeight;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,200;0,400;0,900;1,200;1,400;1,900&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@400;500;600;700&display=swap');

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
  padding: 0 1em;
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
}

.character-text {
  font-family: "Fira Code", monospace;
  color: hsl(0, 40.8%, 60%);
}

.object-text {
  font-family: "Fira Code", monospace;
  color: hsl(248, 40.8%, 60%);
}

.location-text {
  font-family: "Fira Code", monospace;
  color: hsl(82, 40.8%, 60%);
}

.direction-text {
  font-family: "Fira Code", monospace;
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