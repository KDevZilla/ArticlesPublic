

### Introduction
This is a Gomoku/Five in a row game written in C#.
I created this game because of the fond memories I have of playing it on a school computer. 

![UI](https://raw.githubusercontent.com/KDevZilla/ImageUpload/main/SharpMoku/SharpMoku01.png)   

### Overview
Gomoku is a board game where two players take turns placing pieces on a 9x9 or 15x15 board size.  
The goal of this game is to be the first to form a row of five consecutive pieces,  
either horizontally, vertically, or diagonally. 

The winner is the player who succeeds in doing so first.  

### Feature  

- The board can have dimensions of either 9x9 or 15x15.
- Mode Human vs Human, Human vs Bot, Bot vs Human.  
- Bot Level Normal, Hard.  
- It supports both images or colors for the stone and the board.  
- Support various types of themes.  
  
![UI](https://raw.githubusercontent.com/KDevZilla/ImageUpload/main/SharpMoku/SharpMoku02_Small.png)    

### UI
This is a simple Gomoku program so we only need a form that contains one component to render the board.  

### UI\IUI.cs  
This interface defines methods and events. We use FormSharpMoku to implement this interface.  
    
```C#
public interface IUI
    {

        // Raise event CellClicked so the game object will handle the operation
        event Board.CellClickHandler CellClicked;

        void RenderUI();
        void MoveCursorTo(Position position);

        //To tell the game object that the bot has finished moving a cursor to the position to put.
        event EventHandler HasFinishedMoveCursor;


        /*
         These 3 Methods will be trigged from the Game Object.
         Game_GameFinished : UI will display the result.
         Game_BotThinking : UI will change the cursor to an hourglass.
         Game_BotFinishedThinking : Ui will change the cursor back to the default cursor and allow the user to input
        */
        void Game_GameFinished(object sender, EventArgs e);
        void Game_BotThinking(object sender, EventArgs e);
        void Game_BotFinishedThinking(object sender, EventArgs e);
    }
```

### FormSharpMoku.cs  
This form implements IUI interface, it consist PictureBoxGoMoku control.  
   
### UI\PictureBoxGoMoku.cs  

This class inherits from the PictureBox control and consists of a collection of labels used to represent the stone and the board line.  
1. There are a total of 81 labels on a 9x9 board and 225 labels on a 15x15 board.
2. In PictureBoxGoMoKu_Paint, it will only be responsible for rendering the notation and the background image, the rest of the board will be rendered by the labels themself.
The notation of the board can be both Gomoku notation (A-O for the rows, 1-15 for the columns) and the array index position (0-14 for both rows and columns), we use the latter for debugging purposes.

### UI\LabelCustomPaint\GoMokuPaint.cs

To support various kinds of themes, there is more than one class we use to render the label, for the GoMoku theme, we use GoMokuPaint class to render.  
**PaintStone()** This method will render the stone, it also supports rendering an image, or just rendering a stone color.  

**PaintBorder()** This method will draw a line on the label to make the border, it also draws an Intersection point.    
The method handles different kinds of drawing positions.   

TopLeftCorner,  
TopBorder,  
TopRightCorner,  
LeftBorder,  
Center,  
RightBorder  
BottomLeftCorder,  
BottomBorder,  
BottomRightCornder  
  
![UI](https://raw.githubusercontent.com/KDevZilla/ImageUpload/main/SharpMoku/Small/Small_SharpMoku11.png)     
In this image, the blue color is a kind of label, and the green color is the label whose position is Center but also has an Intersection point.  

This is the code that shows only the case of the Center position, 
each position will be drawn in 2 lines and if it is an intersection, there will be a small circle in the center.  

```C#
    private void PaintBorder(Graphics g, ExtendLabel pLabel)
    {
        Point fromPointX = Point.Empty;
        Point toPointX = Point.Empty;
        Point fromPointY = Point.Empty;
        Point toPointY = Point.Empty;


        int beginWidth = 0;
        int middleWidth = pLabel.Width / 2;
        int endWidth = pLabel.Width;
        int beginHeight = 0;
        int middleHeight = pLabel.Height / 2;
        int endHeight = pLabel.Height;

        switch (pLabel.CellAttribute.GoboardPosition)
        {
            case GoBoardPositionEnum.Center:
                fromPointY = new Point(middleWidth, beginHeight);
                toPointY = new Point(middleWidth, endHeight);

                fromPointX = new Point(beginWidth, middleHeight);
                toPointX = new Point(endWidth, middleHeight);
                break;
            //Omit the code for other position
        }


        g.DrawLine(penTable, fromPointY, toPointY);
        g.DrawLine(penTable, fromPointX, toPointX);
        
        g.CompositingMode = CompositingMode.SourceOver;

        if (pLabel.CellAttribute.IsIntersection)
        {
            RectangleF RecCircleIntersecton = new RectangleF(middleWidth - 4, middleHeight - 4, 8, 8);
            g.FillEllipse(ShareGraphicObject.SolidBrush(penTable.Color), RecCircleIntersecton);
        }
    }
```
![UI](https://raw.githubusercontent.com/KDevZilla/ImageUpload/main/SharpMoku/Small/Small_SharpMoku15.png)     
In this image, the blue color is the area of the label,  
BH means beginHeight, EH means endHeight, BW means beginWidth, EW means endWidth.
Between beginHeigh and endHeight is the middleHeight and it is also is also the middleWidth.

For example, the label that has the Center position will need to draw 2 lines.  
1. The horizontal line at the middleHeight from beginWidth to endWidth.  
2. The vertical line at the middleWidth from beginHeight to endHeight.  


**PaintNeighbour()** We don't use this method, but you can uncomment it if you would like it to show the neighbor for debugging purposes.  
![UI](https://raw.githubusercontent.com/KDevZilla/ImageUpload/main/SharpMoku/SharpMoku10.png)  
This image shows the neighbor cell of both black and white stone.  
The purpose of the adjacent position will be discussed in the AI section.


The sequence diagram of when the user clicks.    

![UI](https://raw.githubusercontent.com/KDevZilla/ImageUpload/main/SharpMoku/SharpMoku13.png)      
This is a sequence diagram when the user clicks. I omitted the detail in the (Process) section.  
You can look into the sequence diagram of how to bot works in the Game.cs section.
 

### Gomoku Game
The UI has been completed, and now we will discuss the game component, excluding the AI for now.  

### Board.cs
  We store the value of the stone in a 2D array named Matrix,  and we also use dicWhiteStone, dicBlackStone, and dicNeighbor to store the position of the Black and white stone and their neighbor.  We store the duplicate data to reduce the time to search, supposing we need to know the neighbor position we no need to search through 
all the positions in the Matrix to calculate the neighbor position.
  
```C#
    //Some part of the Board.cs
    [Serializable]
    public class Board
    {

        public delegate void CellClickHandler(object sender, PositionEventArgs positionClick);

        // 2d array to store cell value
        public int[,] Matrix;

        /*
         * dicWhiteStone store WhiteStone position
         * dicBlackStone store WhiteStone position
         * dicNieghbor store position of the stone next to both white and black stone
         */
        public Dictionary<String, SharpMoku.Position> dicWhiteStone { get; private set; } = new Dictionary<String, SharpMoku.Position>();
        public Dictionary<String, SharpMoku.Position> dicBlackStone { get; private set; } = new Dictionary<String, SharpMoku.Position>();
        public Dictionary<String, SharpMoku.Position> dicNeighbor { get; private set; } = new Dictionary<string, Position>();

        public int BoardSize { get; private set; }

        public enum WinStatus
        {
            BlackWon=-1,
            NotDecidedYet=0,
            WhiteWon=1,
            Draw=2
        }

        // Represent the value of the cell in the board
        public enum CellValue
        {
            Black=-1,
            Empty=0,
            White=1,
        }

        public enum Turn
        {
            Black=-1,
            White=1
        }
        public Turn CurrentTurn { get; private set; } = Turn.Black;
        public CellValue CurrentTurnCellValue
        {
            get
            {
                if(CurrentTurn == Turn.Black)
                {
                    return CellValue.Black;
                }

                return CellValue.White;
            }
        }
        public Board (int boardSize)
        {
            if(boardSize != 9 &&
                boardSize != 15)
            {
                throw new ArgumentException($"Board size is invalid {boardSize}, program only accept 9 and 15 as valid value");
            }
            this.BoardSize = boardSize;
            Matrix = new int[this.BoardSize, this.BoardSize];

        }

        public Board(Board board)
        {

            Matrix = new int[board.Matrix.GetLength(0), board.Matrix.GetLength(1)];
            dicWhiteStone = new Dictionary<string, Position>();
            dicBlackStone = new Dictionary<string, Position>();
            dicNeighbor = new Dictionary<string, Position>();
           
            this.Matrix = board.Matrix.Clone() as int[,];
            this.dicWhiteStone = new Dictionary<string, Position>(board.dicWhiteStone);
            this.dicBlackStone = new Dictionary<string, Position>(board.dicBlackStone);
            this.dicNeighbor = new Dictionary<string, Position>(board.dicNeighbor);
            this.listHistory = new List<Position>(board.listHistory);
            this.BoardSize = board.BoardSize;
            this.CurrentTurn = board.CurrentTurn;

        }

        public void PutStone(int pRow, int pCol, CellValue cellValue)
        {
            /* 1.Assign value into the matrix
             * 2.Add the postion value into Hash
             * 3.Add postion into history
             * 4.Add Empty neighbor
             */
            Matrix[pRow, pCol] = (int)cellValue;
            SharpMoku.Position newPosition = new Position(pRow, pCol);
            GetHshByCellValue(cellValue).Add(newPosition.PositionString(), newPosition);
            listHistory.Add(newPosition);
            AddEmptyNeighborOf(newPosition);
        }
```

if we put the black stone into row 0, column 0 using **PutStone(0,0, -1)**  
These things will happen  
1. Set Matrix[0,0] to -1.  
2. Add dicBlackStone with position (0,0) value.
3. Add position(0,0) into a listHistory (program use this value for **Undo()**)  
4. Add dicNeighbor with position(1,0) and position(0,1).  
  
When we call **Undo()**, the program will also need to adjust the neighbor position.  

### Game.cs  
The role of this class is the middle man between the UI and the board object, UI will only raise an event then the game object will decide what to do next,  
All of the business logic will be in the Game and board, the UI only knows how to render the graphic.  
When the user clicks on the board, it will raise an event to the **UI_CellClicked()** method, 
in the **UI_CellClicked()**, the program will check the status of the game and board and then proceed to do **PutStone(positionClick.Value, board.CurrentTurnCellValue)**  

```C#
public class Game
{
     public Board board = null;
     private UI.IUI UI = null;
     public enum GameStateEnum
     {
        NotBegin,
        Playing,
        End
     }
     public enum GameModeEnum
     {
        PlayerVsBot = 0,
        BotVsPlayer = 1,
        PlayerVsPlayer = 2,

     }
     public Board.WinStatus WinResult { get; private set; } = Board.WinStatus.NotDecidedYet;
     public Board.Turn TheWinner { get; private set; }
     public ILog log = null;
     public GameModeEnum GameMode { get; private set; } = GameModeEnum.PlayerVsBot;
     public GameStateEnum GameState { get; private set; } = GameStateEnum.NotBegin;
        /*
         * GameFinished event to tell the UI to display the result
         * BotThinking to tell the UI to change the cursor to an hourglass
         * BotFinishedThinking to tell the UI to move the cursor to the position that Bot needs to put the stone
         */
     public event EventHandler GameFinished;
     public event EventHandler BotThinking;
     public event EventHandler BotFinishedThinking;


     private void ExplicitConstructor(UI.IUI ui,
        Board board,
        int boardSize,
        IEvaluate pbot,
        int botSearchDepth,
        GameModeEnum gameMode)
        {
        this.UI = ui;
        this.GameMode = gameMode;
        this.BotSearchDepth = botSearchDepth;
        if (pbot != null)
        {
            bot = pbot;
        }
        WinResult = Board.WinStatus.NotDecidedYet;

        this.UI.CellClicked -= UI_CellClicked;
        this.UI.HasFinishedMoveCursor -= UI_HasFinishedMoveCursor;
        this.GameFinished -= UI.Game_GameFinished;
        this.BotThinking -= UI.Game_BotThinking;
        this.BotFinishedThinking -= UI.Game_BotFinishedThinking;


        this.UI.CellClicked += UI_CellClicked;
        this.UI.HasFinishedMoveCursor += UI_HasFinishedMoveCursor;

        this.board = (board != null)
            ? board
            : new Board(boardSize);


        this.GameFinished += UI.Game_GameFinished;
        this.BotThinking += UI.Game_BotThinking;
        this.BotFinishedThinking += UI.Game_BotFinishedThinking;

        }
        public Game(UI.IUI ui, Board board,
        IEvaluate pbot,
        int botSearchDepth,
        GameModeEnum gameMode
        )
     {

        ExplicitConstructor(ui, board, 0, pbot, botSearchDepth, gameMode);
       
     }
     public Game(UI.IUI ui, int boardSize,
        IEvaluate pbot,
        int botSearchDepth,
        GameModeEnum gameMode
     )
    {
        ExplicitConstructor(ui, null, boardSize, pbot, botSearchDepth, gameMode);

    }
    public bool CanUndo => board == null
        ? false
        : board.CanUndo;


    private void UI_HasFinishedMoveCursor(object sender, EventArgs e)
    {
        PutStone(botMoveToPostion, (Board.CellValue)board.CurrentTurn);
    }
    public void PutStone(Position position)
    {
        PutStone(position, this.board.CurrentTurnCellValue);
    }
         
    public void PutStone(Position position, Board.CellValue turn)
    {

        board.PutStone(position, turn);
        this.UI.RenderUI();

        WinResult = board.CheckWinStatus();

        if (WinResult == Board.WinStatus.NotDecidedYet)
        {
            
            this.board.SwitchTurn();
            bool IsBotTurn = (GameMode == GameModeEnum.PlayerVsBot && !IsPlayer1Turn) ||
                        (GameMode == GameModeEnum.BotVsPlayer && IsPlayer1Turn);
            if (IsBotTurn)
            {
                BotThinking?.Invoke(this, null);
                BotMove();
            }
            return;
        }

        this.GameState = GameStateEnum.End;
        WinStatusEventArgs statusEvent = new WinStatusEventArgs(WinResult);
        GameFinished?.Invoke(this, statusEvent);


     }

     public void NewGame()
     {
        this.GameState = GameStateEnum.Playing;

        if (this.GameMode == GameModeEnum.BotVsPlayer)
        {
            System.Threading.Thread.Sleep(20);
            BotMove();
        }

     }

     // This method is being used by humans only.
     private void UI_CellClicked(object o, Board.PositionEventArgs positionClick)
     {
        Boolean isPlayerClickDespiteItisBotTurn = (this.GameMode == GameModeEnum.PlayerVsBot && this.board.CurrentTurn != Board.Turn.Black) ||
                                      (this.GameMode == GameModeEnum.BotVsPlayer && this.board.CurrentTurn != Board.Turn.White);

        Boolean isClickedOnNonEmptyCell = board.Matrix[positionClick.Value.Row, positionClick.Value.Col] != (int)Board.CellValue.Empty;
        Boolean isClickedOInValidPosition = !board.IsValidPosition(positionClick.Value);

        if (GameState != GameStateEnum.Playing
            || isClickedOInValidPosition
            || isPlayerClickDespiteItisBotTurn
            || isClickedOnNonEmptyCell)
        {
            return;
        }

        PutStone(positionClick.Value, board.CurrentTurnCellValue);

     }


     public int BotSearchDepth { get; private set; } = 2;
     private Position botMoveToPostion;
     private IEvaluate bot = new EvaluateV3();
     private void BotMove()
     {

        SharpMoku.Board cloneBoard = new Board(this.board);

        Minimax miniMax = new Minimax(cloneBoard, bot, this.log);

        botMoveToPostion = miniMax.calculateNextMove(BotSearchDepth);
        BotFinishedThinking?.Invoke(this, null);
        UI.MoveCursorTo(botMoveToPostion);

	}
}
```

![UI](https://raw.githubusercontent.com/KDevZilla/ImageUpload/main/SharpMoku/SharpMoku14.png)    
This sequence diagram explains the detail part that was omitted from the first diagram.
After the game object called PutStone, if the game result is not decided, 
It will 
1. switch the turn because now it is the bot's turn.
2. Raise the BotThinking event to tell the UI to change the cursor to an hourglass and to block the input from the user.
3. After that call **BotMove()** method, this method will use the Minimax function to find a good position.
4. Raise **BotFinishedThinking** event to tell the UI to change the cursor back to normal.
5. IUI calls MoveCursorTo to mouse your mouse position to the position the bot desires.
6. Raise HasFinishedMouveCurstor to the game object, so it can **PutStone()** by itself.
7. Tell the UI to Render the board.


#### AI
Now come to the AI part, for the search function I use the standard Minimax Alpha-Beta Pruning. 
For the node that the program searches, the program will not search the node that is not a neighbor because the Gomoku board is so big,  
For the 15x15 board size, it has more than 200 positions for the first level, and it will have more than 40,000 positions for the second level Therefore, we aim to minimize the number of nodes by limiting our search to approximately 30 neighbor positions per level.  
We can do this because in Gomoku to win the game, the position that you put must be next to the existing stone position. 
	
#### AI\Minimax.cs  
**board.generateNeighboreMoves()** is the method to get the neighbor position.   
if it is the first level, we allow to get a neighbor at the radius 2 from the existing stone position.  
This class allows you to inject an evaluator object and then call  
**evaluator.evaluateBoard()**  
We separate an evaluator object from the Minimax class because we would like to allow the Minimax to switch between various kinds of evaluator functions.  
As of now we only use **EvaluateV3.cs** but I would like to mention  **EvaluateV2.cs** for education purposes.

```C#
        private MoveScore minimaxSearchAlphaBeta(int depth, SharpMoku.Board board, Boolean IsMax, double AlphaValue, double BetaValue)
        {
            NumberOfNodes++;
            NumberOfNodeInEachLevel[depth]++;
            // Last depth (terminal node), evaluate the current board score.
            String tabString = GetTab(depth);
            MoveScore movescore = new MoveScore();
            Log($"{tabString}depth{depth}");

            if (depth == 0)
            {

                movescore = new MoveScore(evaluator.evaluateBoard(board, !IsMax));
                Log($"{tabString}Evaluate happens here");
                Log($"{tabString}Score::{movescore.Score}");

                return movescore;
            }

            /*If it is first level, the radiusNeighbor can be 2
             * because it will not have too much node.
            */
            int radiusNeighbour = (depth == FirstLevelDepth)
                ? 2
                : 1;
            List<Position> allNeighborPossibleMoves = null;
            if (radiusNeighbour == 2)
            {
                allNeighborPossibleMoves = board.generateNeighboreMoves(radiusNeighbour);
                if (allNeighborPossibleMoves.Count > 30)
                {
                    allNeighborPossibleMoves = board.generateNeighboreMoves(1);
                }
            }
            else
            {
                allNeighborPossibleMoves = board.generateNeighboreMoves(1);
            }


            // If there is no possible move left, treat this node as a terminal node and return the score.
            bool IsNothingLeftToSearch = (allNeighborPossibleMoves.Count == 0);

            if (IsNothingLeftToSearch)
            {
                movescore = new MoveScore(evaluator.evaluateBoard(board, !IsMax));
                return movescore;
            }

            /*If we reach this stage it means
             * There are valid moves
             */

            MoveScore bestMove = new MoveScore();
            int depthChild = 0;
            Boolean isMaxChild = false;
            depthChild = depth - 1;
            isMaxChild = !IsMax;

            bestMove.Row = allNeighborPossibleMoves[0].Row;
            bestMove.Col = allNeighborPossibleMoves[0].Col;
            bestMove.Score = IsMax
                            ? int.MinValue
                            : int.MaxValue;
            int iCountMove = 0;
            Log($"{tabString}No of neighbor::{allNeighborPossibleMoves.Count }");
            foreach (Position move in allNeighborPossibleMoves)
            {

                iCountMove++;
                Log($"{tabString}{iCountMove}.   move::{move.PositionString()}");
                board.PutStoneAndSwitchTurn(move);
                movescore = minimaxSearchAlphaBeta(depthChild, board, isMaxChild, AlphaValue, BetaValue);
                movescore.Row = move.Row;
                movescore.Col = move.Col;

                Log($"{tabString}Score::{movescore.Score }");
                //  board.Undo();

                if (board.IsFull)
                {
                    Log("{tabString}board.IsFull");
                    return movescore;

                }
                board.Undo();

                if (IsMax)
                {
                    AlphaValue = Math.Max(movescore.Score, AlphaValue);
                    if (movescore.Score >= BetaValue)
                    {
                        Log($"{tabString}moveScoe >= Beta");
                        return movescore;
                    }
                    bestMove = MoveScore.Max(bestMove, movescore);

                }
                else
                {
                    BetaValue = Math.Min(movescore.Score, BetaValue);
                    if (movescore.Score > AlphaValue)
                    {
                        Log($"{tabString}moveScore > Alpha");
                        return movescore;
                    }
                    bestMove = MoveScore.Min(bestMove, movescore);

                }
            }
            return bestMove;

        }
```

#### AI\EvaluateV1.cs  
This class just chooses a random position.  

#### AI\EvaluateV2.cs
This is the popular Gomoku evaluator function, the idea of this evaluator function is we try to search the whole board to find the pattern in 
Horizontal, Vertical, and Diagonal direction.  
![AI](https://raw.githubusercontent.com/KDevZilla/ImageUpload/main/SharpMoku/Small/Small_SharpMoku12.png)   

For a 15x15 dimensions board, there will be 88 lines.
15 lines from Horizontal.   
15 lines from Vertical.  
58 lines from both directions of the diagonal each direction has 29 lines.  

In each line we search to find the pattern, the more consecutive stones we have, the higher score we get unless 
our pattern was blocked by the opponent's stone.  
To give the score, who is the current is also needs to be accounted for, for example, if we have 4 stones in a row and now it is 
our turn, we can guarantee that we will win but if the current turn is the opponent, we will not get much benefit from this pattern because the opponent 
can block us from getting the winning position.  

![AI](https://raw.githubusercontent.com/KDevZilla/ImageUpload/main/SharpMoku/SharpMoku16.png)     
These 3 lines have 4 stones in a row, the difference is  
Line #15 has no block from the opponent.   
Line #13 has one block on the left from the opponent.  
Line #11 has 2 blocks on both left and right from the opponent.  


I will use   
**X** to represent our stone,  
**-** to represent the blank space,   
**O** to represent the opponent's stone.   

The Score in this table is not an extract value, they are just an idea, that can be adjusted.   
The function only needs Pattern, Whose turn is as the parameters,  
I show the Number of block columns on this table just to make it easier to look at.  

|Pattern|Number of blocks|Whose turn|Score|Description|
|-------|------------|------|-----|-----------|
|XXXXX|0|(N/A)|50000000|5 in a row. We can win the game |
|-XXXX-|0|Our turn |1000000|4 in a row with no block, can confirm to win because it has 4 stones already and this is my turn|
|OXXXX-|1|Our turn |1000000|4 in a row with one  block on the left, can confirm to win because it has 4 stones already and this is my turn|
|OXXXX-|1|Opponent turn |1000|4 in a row with one  block, but it is the opponent's turn, so our opponent can block us|
|-XXX-|0|Our turn|200|3 in a row with no block, this is not bad it has the potential to win  |
|OXXXXO|2|(N/A)|0|4 in a row with 2 blocks from the left and right. This is useless because our pattern was blocked by the opponent's stone at both sides |  

This evaluation algorithm is o.k. I can play with it, but when I increase the depth of the bot, the game is too slow.   
I found 2 issues with this pattern   
1. The number of the cells we check is too huge, there are about 900 cells from 225 + 225 + 450 (horizontal, vertical, 2 directions of diagonal)  
2. This algorithm does not give much score for patterns like this XXX-XO compared to XXXX-O, both patterns can be won by putting a single stone, but for the first pattern,  
   the algorithm just sees it as 2 patterns of 2 consecutive stones.  
   XXX-XO and XXX------XO have the same score despite the first one can make us win by a single stone.

#### AI\EvaluateV3.cs  
Since EvaluateV2 is not good enough so I tried to search for another solution, I found the javascript GoMoku program by Anton Midrenok  
https://codepen.io/mudrenok/pen/gpMXgg  
This EvaluateV3 is a port to C# with the reflection at some part of the code   
The idea of this function is, that when the program evaluates, it does not need to scan the whole board, it just needs to search 36 cells from the position it wants to put the stone.  
Supposing we would like to know the score of the position 7,7  
These are the positions that it will search  
![AI](https://raw.githubusercontent.com/KDevZilla/ImageUpload/main/SharpMoku/SharpMoku08.png)      

There are 4 directions,  
North to South  for Vertical.  
West to East   for Horizontal.  
NorthEast to SouthWest and NorthWest to SouthEast for both diagonals.   

Each direction will search for 9 cells only, the position itself, and the other 8 cells in the line.
In each line, we search for these kinds of patterns.  

These are examples of the patterns and the scores.  
* Some of the score values are "Depend on how many and another factor" Please look into the **getScoreByPattern()** for more understanding.
  
|Pattern Name|Sample of Pattern|Score|
|-------|------------|------|
|Stone5|XXXXX|1000000000|
|Stone4WithNoBlock |-XXXX-|100000000|
|Stone3WithNoBlock|-XXX--,--XXX-,-X-XX-,-XX-X-|10000000|
|Stone2WithNoBlock|--XX--,-X-X--,--X-X-,-XX---,---XX-,-X--X-|Depend on how many and another factor|
|Stone4WithBlock|OX-XXX,OXX-XX,OXXX-X,OXXXX-,-XXXXO,X-XXXO,XX-XXO,XXX-XO,|Depend on how many and another factor|
|Stone3WithBlock|OXXX--,OXX-X-,OX-XX-,--XXXO,-X-XXO,-XX-XO,|Depend on how many and another factor|  

  
**GetListAllDirection** This function will get the list of patterns from 4 directions.  
**GetCellValueInDirection()** This function will get the pattern from the put position(positionCheck)  

Supposing you need to check the pattern on the position row 0, column 6. West to East direction.
There are 3 steps.  
1. First loop check 4 cells  [0,5],[0,4],[0,3],[0,2] then insert into listCell.  
2. Add 0,6 cell value into the list.   
3. Second loop check 4 cells  [0,7][0,8],[0,9],[0,10] then add into listCell.  
The reason why the first loop we insert at the 0 position is  
we would like to have the data like this  
		2,3,4,5,6,7,8,9,10  
The order of the first loop is 5,4,3,2 but we need to get 2,3,4,5 so  
We insert it at the 0 position so that we can get 2,3,4,5  
For the second loop, its sequence is 6, 7, 8, 9, 10 which is already what we want.    
![AI](https://raw.githubusercontent.com/KDevZilla/ImageUpload/main/SharpMoku/SharpMoku09.png)   
This image shows the position of the cell that needs to be checked.  

**getScoreByPattern** This function will calculate the score by a giving pattern.  

```C#
public List<List<int>> GetListAllDirection(SharpMoku.Board board, Position checkPosition, SharpMoku.Board.CellValue cellValue)
        {
            Position positionDeltaNorthSouth = new Position(1, 0);
            Position positionDeltaWestEast = new Position(0, 1);
            Position positionDeltaNorthWest = new Position(1, 1);
            Position positionDeltaNorthEast = new Position(1, -1);

            /*
                   *Prepare to go though all 8 directions
                   * 4 have 4 lists of News because each list go both way
                   * For example NorthSouth mean from the position to north and from the postion to south
                   */
            List<int> listNorthSouth = GetCellValueInDirection(board.Matrix, cellValue, checkPosition, positionDeltaNorthSouth);
            List<int> listWestEast = GetCellValueInDirection(board.Matrix, cellValue, checkPosition, positionDeltaWestEast);
            List<int> listNorthWest = GetCellValueInDirection(board.Matrix, cellValue, checkPosition, positionDeltaNorthWest);
            List<int> listNorthEast = GetCellValueInDirection(board.Matrix, cellValue, checkPosition, positionDeltaNorthEast);

            List<List<int>> listAllDirection = new List<List<int>>()
            {
                listNorthSouth ,
                listWestEast ,
                listNorthWest ,
                listNorthEast
            };

            return listAllDirection;
        }

public List<int> GetCellValueInDirection(int[,] matrix, SharpMoku.Board.CellValue cellValue, Position positionCheck, Position positionDelta)
        {
            int i;
            List<int> listCell = new List<int>();
            bool IsCheckPostionIsNotmatchWithCellValue = matrix[positionCheck.Row, positionCheck.Col] != (int)cellValue;
            HashSet<String> hshCellInaRow = new HashSet<string>();
            if (IsCheckPostionIsNotmatchWithCellValue)
            {
                return listCell;
            }
            int opponentCellvalue = -(int)cellValue;
            //First loop Insert cell #1
            for (i = 1; i < 5; i++)
            {
                Position nextPosition = new Position(positionCheck.Row - positionDelta.Row * i,
                                                    positionCheck.Col - positionDelta.Col * i);
                if (nextPosition.Row < 0 ||
                    nextPosition.Row >= matrix.GetLength(0) ||
                    nextPosition.Col < 0 ||
                    nextPosition.Col >= matrix.GetLength(0))
                {
                    break;
                }

                var nextValue = matrix[nextPosition.Row, nextPosition.Col];
                if(!hshCellInaRow.Contains ( nextPosition.PositionString()))
                {
                    listCell.Insert(0, nextValue); //We insert at the 0 position
                }
                
                if ((int)nextValue == opponentCellvalue)
                {

                    break;
                }

            }
            listCell.Add((int)cellValue); //The cell itself #2

            //Add #3
            for (i = 1; i < 5; i++) 
            {
                Position nextPosition = new Position(positionCheck.Row + positionDelta.Row * i,
                                                    positionCheck.Col + positionDelta.Col * i);
                if (nextPosition.Row < 0 ||
                    nextPosition.Row >= matrix.GetLength(0) ||
                    nextPosition.Col < 0 ||
                    nextPosition.Col >= matrix.GetLength(0))
                {
                    break;
                }
                var nextValue = matrix[nextPosition.Row, nextPosition.Col];

                if (!hshCellInaRow.Contains(nextPosition.PositionString()))
                {
                    listCell.Add(nextValue);//We add it to the last position
                }
                if ((int)nextValue == opponentCellvalue)
                {
                   // listCell.Insert(0, nextValue);
                    break;
                }
                //listCell.Insert(0, nextValue);
            }
            return listCell;
        }

public int getScoreByPattern(NumberofScorePattern numberofPattern)
        {
            if (numberofPattern.Winning > 0)
            {
                return CONST_winScore * numberofPattern.Winning;
            }
            if (numberofPattern.Stone4 > 0)
            {
                return CONST_winGuarantee;

            }

            if (numberofPattern.BlockStone4 > 1)
            {
                return CONST_winGuarantee / 10;
            }
            if (numberofPattern.Stone3 > 0
                && numberofPattern.BlockStone4 > 0)
            {
                return CONST_winGuarantee / 100;
            }
            if (numberofPattern.Stone3 > 1)
            {
                return CONST_winGuarantee / 1000;
            }

            if (numberofPattern.Stone3 == 1)
            {
                switch (numberofPattern.Stone2)
                {
                    case 3: return 40000;
                    case 2: return 38000;
                    case 1: return 35000;
                    default: return 3450;
                }
            }

            if (numberofPattern.BlockStone4 == 1)
            {
                switch (numberofPattern.Stone2)
                {
                    case 3: return 4500;
                    case 2: return 4200;
                    case 1: return 4100;
                    default: return 4050;
                }
            }


            switch (numberofPattern.BlockStone3)
            {
                case 3:
                    if (numberofPattern.Stone2 == 1) return 2800;
                    break;
                case 2:
                    switch (numberofPattern.Stone2)
                    {
                        case 2: return 3000;
                        case 1: return 2900;
                    }
                    break;
                case 1:
                    switch (numberofPattern.Stone2)
                    {
                        case 3: return 3400;
                        case 2: return 3300;
                        case 1: return 3100;
                    }
                    break;
            }

            switch (numberofPattern.Stone2)
            {
                case 4: return 2700;
                case 3: return 2500;
                case 2: return 2000;
                case 1: return 1000;
            }
            return 0;
        }
```
This evaluation function is very strong and it solves 2 issues that Evaluate2 has
1. The number of cells we check is not too huge anymore.
2. This algorithm is better at handling the pattern like this XXX-XO

### Testing 
You can just run the scripts from Visual Studio.  
![Test Result](https://raw.githubusercontent.com/KDevZilla/ImageUpload/main/SharpMoku/SharpMoku99.png)    

### What can we do to improve  
For the UI, If I need to rewrite the board object again, I might consider not using the array of labels to render the board and the stone.  
It might be better if all of the objects we see on the board just be painted by a single picturebox object.  

For the AI, it is already strong enough, but it can be stronger if we implement some of the opening algorithms and also use Zobrist hash.  


### Reference  
https://en.wikipedia.org/wiki/Gomoku  
https://blog.theofekfoundation.org/artificial-intelligence/2015/12/11/minimax-for-gomoku-connect-five/  
https://codepen.io/mudrenok/pen/gpMXgg  




                                                                                    


