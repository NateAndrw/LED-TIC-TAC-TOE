// PROJECT  : LED TIC TAC TOE
// PURPOSE  : LED Tic Tac Toe using ATmega328P and TPIC6C595 shift registers
// MCU      : ATmega328P (Standalone)
// AUTHOR   : N.ANDREW
// DATE     : 2025-01-18
// STATUS   : PCB Version

// Pin definitions for shift registers
const int latchPin = PD4; // RCK
const int dataPin = PD2;  // SERIN
const int clockPin = PD3; // SRCK

// Button pins
const int onOffSwitch = A0;  // ADC0
const int resetButton = A1;  // ADC1
const int switchButton = A2; // ADC2
const int selectButton = A3; // ADC3

// Game state variables
int gameBoard[9] = {0}; // 0 = open, 1 = Player 1 (red), 2 = Player 2 (green)
int currentPlayer = 1;  // 1 = Player 1, 2 = Player 2
int selectedTile = 0;   // The currently selected tile (flashing)
int movesCount = 0;     // Number of moves made in the game
bool gameEnded = false; // Flag to indicate if the game has ended
int winner = 0;         // 0 = None, 1 = Player 1, 2 = Player 2

// Flashing LED variables
bool flashState = false;        // Current state of flashing LED
unsigned long lastFlashTime = 0;
const unsigned long flashInterval = 500; // Flashing interval in ms

// Function declarations
void resetGame();
void handleInput();
void updateLEDs();
byte getLEDData(int tile);
void checkGameStatus();
void flashSelectedTile();
void printGameState();

void setup() {
  // Set pin modes
  pinMode(latchPin, OUTPUT);
  pinMode(dataPin, OUTPUT);
  pinMode(clockPin, OUTPUT);
  pinMode(resetButton, INPUT_PULLUP);
  pinMode(selectButton, INPUT_PULLUP);
  pinMode(switchButton, INPUT_PULLUP);
  pinMode(onOffSwitch, INPUT_PULLUP);

  // Initialize serial monitor
  Serial.begin(9600);
  Serial.println("LED Tic Tac Toe Initialized");

  // Start game
  resetGame();
}

void loop() {
  // Check if the game is powered on
  if (digitalRead(onOffSwitch) == LOW) {
    Serial.println("Game powered off.");
    delay(1000);
    return;
  }

  // Handle input from buttons
  handleInput();

  // Handle flashing for the selected tile if the game is not ended
  if (!gameEnded) {
    flashSelectedTile();
  }
}

// Resets the game to its initial state
void resetGame() {
  // Reset game variables
  for (int i = 0; i < 9; i++) {
    gameBoard[i] = 0;
  }
  currentPlayer = 1;
  selectedTile = 0;
  movesCount = 0;
  flashState = false;
  gameEnded = false;
  winner = 0; // Reset winner

  // Update LEDs and print to Serial
  updateLEDs();
  Serial.println("Game reset. Player 1 (red) starts.");
  printGameState();
}

// Handles button input
void handleInput() {
  if (digitalRead(resetButton) == LOW) {
    resetGame();
    delay(200); // Debounce
    return;
  }

  if (digitalRead(switchButton) == LOW) {
    // Move to the next open tile
    do {
      selectedTile = (selectedTile + 1) % 9;
    } while (gameBoard[selectedTile] != 0);

    Serial.print("Tile switched to: ");
    Serial.println(selectedTile + 1);
    delay(200); // Debounce
  }

  if (digitalRead(selectButton) == LOW) {
    if (gameBoard[selectedTile] == 0) {
      // Place marker for the current player
      gameBoard[selectedTile] = currentPlayer;
      movesCount++;

      Serial.print("Player ");
      Serial.print(currentPlayer);
      Serial.print(" selected tile: ");
      Serial.println(selectedTile + 1);

      // Check for win/tie conditions
      checkGameStatus();

      // Switch to the next player if the game has not ended
      if (!gameEnded) {
        currentPlayer = (currentPlayer == 1) ? 2 : 1;

        // Move to the next open tile
        if (movesCount < 9) {
          do {
            selectedTile = (selectedTile + 1) % 9;
          } while (gameBoard[selectedTile] != 0);
        }

        // Update LEDs and print game state
        updateLEDs();
        printGameState();
      }
    } else {
      Serial.println("Tile already chosen. Choose another.");
    }
    delay(200); // Debounce
  }
}

// Updates the LED board based on the game state
void updateLEDs() {
  byte shiftRegisterData[3] = {0}; // Data for the 3 shift registers

  if (gameEnded && winner != 0) {
    // Display winner's color on the whole board
    byte color = (winner == 1) ? 0b01010101 : 0b10101010; // Red or Green
    for (int i = 0; i < 3; i++) {
      shiftRegisterData[i] = color;
    }
  } else {
    for (int i = 0; i < 9; i++) {
      int regIndex = i / 3;      // Determine which shift register
      int bitIndex = (i % 3) * 2; // Determine bit position in register

      if (gameBoard[i] == 1) {
        shiftRegisterData[regIndex] |= (1 << bitIndex); // Red on
      } else if (gameBoard[i] == 2) {
        shiftRegisterData[regIndex] |= (1 << (bitIndex + 1)); // Green on
      }
    }

    // Handle flashing LED
    if (flashState && gameBoard[selectedTile] == 0 && !gameEnded) {
      int regIndex = selectedTile / 3;
      int bitIndex = (selectedTile % 3) * 2;
      shiftRegisterData[regIndex] |= (currentPlayer == 1) 
                                      ? (1 << bitIndex)        // Red flashing
                                      : (1 << (bitIndex + 1)); // Green flashing
    }
  }

  // Send data to shift registers
  digitalWrite(latchPin, LOW);
  for (int i = 2; i >= 0; i--) {
    shiftOut(dataPin, clockPin, MSBFIRST, shiftRegisterData[i]);
  }
  digitalWrite(latchPin, HIGH);
}

// Toggles flashing for the selected tile
void flashSelectedTile() {
  if (millis() - lastFlashTime >= flashInterval) {
    lastFlashTime = millis();
    flashState = !flashState;
    updateLEDs();
  }
}

// Checks for a win or tie condition
void checkGameStatus() {
  const int winPatterns[8][3] = {
    {0, 1, 2}, {3, 4, 5}, {6, 7, 8}, // Rows
    {0, 3, 6}, {1, 4, 7}, {2, 5, 8}, // Columns
    {0, 4, 8}, {2, 4, 6}             // Diagonals
  };

  // Check for a winner
  for (int i = 0; i < 8; i++) {
    int a = winPatterns[i][0];
    int b = winPatterns[i][1];
    int c = winPatterns[i][2];
    if (gameBoard[a] != 0 && gameBoard[a] == gameBoard[b] && gameBoard[b] == gameBoard[c]) {
      Serial.print("Player ");
      Serial.print(gameBoard[a]);
      Serial.println(" wins!");
      gameEnded = true;
      winner = gameBoard[a]; // Set the winner
      updateLEDs();
      printGameState();
      return;
    }
  }

  // Check for a tie
  if (movesCount == 9) {
    Serial.println("Game over: It's a tie!");
    gameEnded = true;
    printGameState();
  }
}

// Prints the current game state to the serial monitor
void printGameState() {
  char symbols[3] = {'-', 'X', 'O'}; // Symbols for open, Player 1, Player 2
  for (int i = 0; i < 9; i++) {
    Serial.print(symbols[gameBoard[i]]);
    if ((i + 1) % 3 == 0) {
      Serial.println();
    } else {
      Serial.print(" | ");
    }
  }
  Serial.println();
}
