![Animation](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/KReversi_Animaton_2022_11_20.gif)   

# Introduction
There are many Reversi programmes already, and I chose to develop this programme   
because I couldn't find one with the features I needed when looking for a Revesi/Othell game.  
  
# Features 
1. Support Mode Human vs Human, Human vs Bot, Bot vs Bot.
2. Support board editor. 
3. Support Bot creator. You can choose an image and customize an AI score
		that will be used to evaluate the board.
		
4. Can show the bot's last move Minimax search tree.
5. Can change the profile picture of Human Player1 and Human Player2
6. Can navigate the move.
7. Support Dark Mode.

## File Extensions used by the program.
1. .brd is used as board information.
2. .bot is used as bot information. 
3. .rev is used as game save information, Game information retains the history of the move,   
which is what distinguishes it from a board game, so you can navigate it via the navigate control.

## How to play ##
The rule of the game is exactly the same as a normal Reversi game.


The Concept
## File Structure of the project ##  
![File Structure](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/Small/FileStructure.PNG)   

## Graphic  
In this section, I would like to talk about the classes responsible to render the graphic  
but I will not mention anything related to the Board Editor or Bot Creator yet.  

The Game Board

![UIControl01](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/UIControl01.png)  



![UIControl01](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/UIControl02.png)  
The board also can display number.
### UI\PictureBoxBoard.cs 
This is a class that inherits from PictureBox control.  
It is responsible to display the Reversi board. This class itself consists of the control called   pictureBoxTable.  
the pictureBoxTable control will be the one that renders the table can disk  
from the pictureBoxTable_Paint() method.  
The pictureBoxTable_Paint() method read the board information from this class to  
draw the table and disk.  
		
These are the logic for drawing the board.  
 1. There is an array of 64 Rectangles, we create these 64 Rectangles to 
store the position of the cells on the board.  
 2. This method will call  
  **DrawBoard()** to draw the board table.  
  **DrawIntersection()** as its names suggest.  
  **DrawDisk()** just draw disk.  
  **DrawRedDot()** to show which is the last put position.  
  **DrawDiskBorder()** to show which cell can be put.  
  **DrawNumber()** this method is for when we need to display the score of the cell.  
  All of the drawings in this method just use basic GDI+ methods() such as  
    **FillEllipse()**
    **DrawEllipse()**
    **DrawRectangles()**  
			
    There are no images being used for board drawing.
These are the logic for drawing the notation
     1. Just loop from 0 to 7  
     2. using **DrawString()** method to draw the string  
This control has **PictureBoxBoard_Paint()** method to render the notation.  



```C#
 public enum CellImageEnum
 {
    WhiteCell,
    BlackCell,
    BlankCell,
    BlankCellWithRed  //To show which is the last cell that the disk was put
 }
 public enum BoardModeEnum
 {
    PlayMode,
    EditMode
 }
 public Boolean IsSmallBoard { get; set; } = false; // Suppot display the board as small size
 public Boolean IsHideLegent { get; set; } = false; 
 public Boolean IsDrawNotion { get; set; } = true;  // Can hide the notation (A1-H8)

 public event PictureBoxCellClick CellClick;
 private void PictureBox1_MouseDown(object sender, MouseEventArgs e)
 {
    //Must be a left button
    if (e.Button != MouseButtons.Left)
    {
        return;
    }

    int X = e.X;
    int Y = e.Y;
    int RowClick = Y / CellSize;
    int ColClick = X / CellSize;
    if (RowClick >= NoofRow ||
        ColClick >= NoofRow ||
        RowClick < 0 ||
        ColClick < 0)
    {
        //Row and Column click must be in range.
        return;
    }

    if (CellClick != null)
    {
        //Raise event.
        CellClick(this, new Position(RowClick, ColClick));
    }

 }
 private void DrawNumber(Graphics g, int Number, Rectangle rec)
 {
     /*This method is for drawing a number in case of 
       showing a score value from evaluate function.
     */
     Rectangle r = rec;
     r.Height -= 10;
     r.Width -= 10;
     r.X += 10;
     r.Y += 15;

     Font SegoeUIFont = new Font("Segoe UI", 14, FontStyle.Bold);
     Color NumberColor = Color.FromArgb(150, 180, 180); 
     String NumberText = "+" + Number;
     if(Number < 0)
     {
       //Negative number does not need "+" prefix.
       NumberText =  Number.ToString ();
       NumberColor = Color.FromArgb(255, 199, 79);
     }
     //Create Brush and other objects
     PointF pointF = new PointF(r.X, r.Y);
     SolidBrush solidBrush = new SolidBrush(NumberColor );
     //Draw text using DrawString
     g.DrawString(NumberText , 
       SegoeUIFont,
       solidBrush, pointF );

     solidBrush.Dispose();

 }
 private void DrawDisk(Graphics g, CellImageEnum CellImage, Rectangle rec)
 {

     if (CellImage == CellImageEnum.BlankCell)
     {
       return;
     }
     Rectangle r = rec;
     //Reduce the size of Rectangle
     r.Height -= 10;
     r.Width -= 10;
     r.X += 5;
     r.Y += 5;
     DrawDiskBorder(g, r);

     Color colorDisk = Color.White;
     if (CellImage != CellImageEnum.WhiteCell)
     {
       colorDisk = Color.Black;
     }

     using (Brush brushDisk = new SolidBrush(colorDisk))
     {
       g.FillEllipse(brushDisk, r);
     }

     
     Color diskBorderColor = Color.Black;

     //Uncomment this line in case you would like white disk to have white color border
     //diskBorderColor = colorDisk
     using (Pen penDisk = new Pen(new SolidBrush(diskBorderColor)))
     {
      g.DrawEllipse(penDisk, r);
     }

 }
 private void DrawDiskBorder(Graphics g, Rectangle rec)
 {
     //Reduce the size of Rectangle
     Rectangle r = rec;
     r.Height -= 10;
     r.Width -= 10;
     r.X += 5;
     r.Y += 5;
     using (Pen penDisk = new Pen(DiskBorderColor))
     {
       g.DrawEllipse(penDisk, r);
     }
 }
 private void DrawBoard(Graphics g, Rectangle[] arrRac)
 {
     //This method just draw 64 array of Rectangle objects.

     g.Clear(BoardColor);
     g.DrawRectangles(PenBorder, arrRac);
     Rectangle rectangleBorder = new Rectangle(0, 0, CellSize * 8, CellSize * 8);
     using(Pen penBigBorder=new Pen ( Color.Black, 4))
     {
       g.DrawRectangle(penBigBorder, rectangleBorder);
     }
 }
```

### The Board Information  
  The Board information will display the picture of the Player and Name and the number of disks for each side    They are just a group of controls on FormGame, that will be used in the ReverseUI class.
	
### The Navigator control and the history of the moves.
  These are just controls in FormGame that will be used in ReverseUI class.  
  The Navigator control consists of 4 buttons, the move history is a tableLayoutPanel that  
  contain a  Link label.

	
### Theme.cs  
This class just contains the control color information.

```C#
    public class Theme
    {
        public Color ButtonBackColor { get; set; }
        public Color ButtonForeColor { get; set; }
        public Color LabelForeColor { get; set; }
        public Color FormBackColor { get; set; }
        public Color InputBoxBackColor { get; set; }
        public Color LinkLabelForeColor { get; set; }
        public Color LinkLabelActiveForeColor { get; set; }
        public Boolean IsFormCaptionDarkMode { get; set; } = true;

        public Color MenuBackColor { get; set; }
        public Color MenuHoverBackColor { get; set; }
        public Color MenuForeColor { get; set; }

    }
```

  This Program supports DarkMode, we use Global class to have LightTheme, DarkTheme  
Then, when each of the forms loads, it will access Global.CurrentThem then uses themeUtil  
to set the appearance of the controls.  
  
  This is an example of using themUtil object.  
  

```C#	
themeUtil.SetLabelColor(theme, lblNumberofBlackDisk,
                        lblNumberofWhiteDisk,
                        lblPlayer1Name,
                        lblPlayer2Name)
         .SetButtonColor(theme, btnFirst,
                        btnNext,
                        btnPrevious,
                        btnLast)
         .SetMenu (this.menuStrip1)
         .SetForm(this);
```
### FormGame.cs 
  This class is the main form, it contains the **PictureBoxBoard**  
  and the **Navigator** controls and history of moves.  
### UI.Dialog.cs
  All of the code to show dialog belongs here.  
	
## The Important Classes
### AI.Board.cs
This is the class that holds the board information  
 Fields and Properties  
   **int[,] boardMatrix** we use simply 2d array to contain the disk, **Black =-1, Blank =0, White =1**  
   **BoardPhase** has three phases: Beginning, Middle, and EndGame.  
   This value will be used by AI, it will determine  
   which set of score values it needs to evaluate the board score.  
   **NumberofLastFlipCell** every time we flip the cell, we keep the number of the cells that were flipped.  
   **CurrentTurn** the current turn.  
 Methods
   **generateMoves()** generates available moves.  
   **PutValue()** just put the cell value.  
   **IsLegalMove()** check to see if the position is valid  
   **SwitchTurn()** just switch the turn.  
   **IsTherePlaceToPut()** this method will return false if there is no place to put it; otherwise, it will return true.  

These are the Board constructors.
```C#
        public Board()
        {
            boardMatrix = new int[8, 8];
            SetCell(3, 3, CellValue.White);
            SetCell(3, 4, CellValue.Black);
            SetCell(4, 3, CellValue.Black);
            SetCell(4, 4, CellValue.White);
            listPutPosition.Clear();
        }

        public Board(Board OriginalBoard)
        {
            boardMatrix = new int[8, 8];
            Array.Copy(OriginalBoard.boardMatrix, this.boardMatrix, this.boardMatrix.Length);
            this.CurrentTurn = OriginalBoard.CurrentTurn;
            if (OriginalBoard.LastPutPosition != null)
            {
                this.SetLastPutPosition(OriginalBoard.LastPutPosition.Clone());
            }
        }
```
### AI.BoardValue.cs  
  This class contains the board information, when we would like to save the board or create a custom board,   
We store the value in this class and then Serialized it.   


### ReverseUI.cs
   this class implements the IUI interface, so it must have these methods.   
  
```C#
	void RenderHistory();
        void RenderNumberofDisk(int WhiteDisk, int BlackDisk);
        void ShowBoardAtTurn(int NumberofTurn);
        void MoveBoardToNextTurn(); 
        void MoveBoardToPreviousTurn();
        void RenderBoard();
        void SetGame(KReversiGame game);
        void RemoveGame();
        void BlackPutCellAt(Position position);
        void WhitePutCellAt(Position position);
        void Initial();
        void ReleaseUIResource(); //To make sure there is no event leak
        void InformPlayer1NeedtoPass(); 
        void InformPlayer2NeedtoPass();
        void InformGameResult(KReversiGame.GameResultEnum result);
        
	
	// Any event the begin with MoveBoard relate to navigation.
        event EventHandler MoveBoardToNextTurnClick;
        event EventHandler MoveBoardToPreviousTurnClick;
        event EventHandler MoveBoardToFirstTurnClick;
        event EventHandler MoveBoardToLastTurnClick;
        event PictureBoxBoard.PictureBoxCellClick CellClick;
        event EventInt MoveBoardToSpecificTurnClick;
        event EventHandler ContinuePlayingClick;
```
### KReversiGame.cs
This class contains **UIU** object and board object it works as a glue between   
these 2 objects.  
  This program use Inversion of control  
  https://en.wikipedia.org/wiki/Inversion_of_control  
  It means the UI part will not know anything much about the game data when we click on a cell   
  it will not check if this cell is empty or if it is a valid position or not, it will just notify   
  the game that this specific cell was clicked, then the game object will determine what to do next.  
  For example, the game object will check if it is a valid position and then   
  it will call a board object to update the value and then   
It will notify the UI to let them know that it needs to update or it will call the UI to update itself   directly.  


Both **BotPlayer1MoveDecision()** and   
**HumanPlayer1MoveDecision()**  
will call Player1Move(). The difference is BotPlayer1MoveDecision will   
call minimaxbot.MakeMove(board) to get the bot position, while 
**HumanPlayer1MoveDecision()** will get the position from user input.

**Player1Move()**
	- verify if the position is valid.  
	- Put the disk then switch turn.  
	- handle in case of passes.  
	- store board to history.  
	- Tell UI to render a board.  
	- Check if the game result is finished.  
 

```C#
public enum PlayerMode
{
    FirstHuman_SecondHuman = 0,
    FirstHuman_SecondBot = 1,
    FirstBot_SecondHuman = 2,
    FirstBot_SecondBot = 3
}
public enum GameStatusEnum
{
    Pause=-1,
    Playing = 0,
    Finished = 1
}

public enum GameResultEnum
{
    NotDecideYet=-2,
    BlackWon = -1,
    Draw = 0,
    WhiteWon = 1,
}

public void BotPlayer1MoveDecision(out bool CanMove)
{
	CanMove = false;
	if (this.Player1 == null)
	{
		throw new Exception("Please Assign Player1bot");
	}
	if (this.GameState == GameStatusEnum.Finished ||
		this.GameState == GameStatusEnum.Pause)
	{
		return;
	}
	if (!IsPlayer1Bot)
	{
		return;
	}
	if (IsPlayer1NeedtoPass())
	{
		//This player1 is bot no need to inform
		//UIBoard.InformPlayer1NeedtoPass();
		board.SwitchTurnDueToPlayerPass();
	}
	else
	{
		Player1BotBeginToThink?.Invoke(this, new EventArgs()); // To make UI change mouse cursor to sandy clock
		if (Player1 is MiniMaxBotProto)
		{
			MiniMaxBotProto minimaxBot = (MiniMaxBotProto)Player1;
			// minimaxBot.
			//For Player1 IsKeepLastDecisionTree is always false;
			minimaxBot.IsAllowRandomDecision = this.IsAllowRandomDecision;
			minimaxBot.IsKeepLastDecisionTree = false;// this.IsKeepLastDecisionTree;
			minimaxBot.IsUsingAlphaBeta = this.IsUsingAlphaBeta;
		}
		Position botPosition = Player1.MakeMove(board);
		Player1Move(botPosition, out CanMove);
		if (!CanMove)
		{
			throw new Exception("There must be something wrong with Player 1 Move");
		}
		Player1BotFinishedToThink?.Invoke(this, new EventArgs()); // To make UI change mouse cursor back
	}
	OnPlayer1Moved();
}
public void HumanPlayer1MoveDecision(Position pos, out bool CanMove)
{
	CanMove = false;
	if (this.GameState == GameStatusEnum.Finished ||
		this.GameState == GameStatusEnum.Pause)
	{
		return;
	}
	if (IsPlayer1Bot)
	{
		return;
	}

	Player1Move(pos, out CanMove);

	OnPlayer1Moved();

}
private void Player1Move(Position pos, out Boolean CanMove)
{
	CanMove = false;
	if (this.board.CurrentTurn != Player1Color)
	{
		return;
	}
	if (!this.board.IsLegalMove(pos, Player1Color))
	{
		return;
	}


	board.PutAndAlsoSwithCurrentTurn(pos, this.Player1Color);
	CanMove = true;
	boardHistory.IndexMoveAdd();
	Boolean CanPlayer2MakeMove = board.generateMoves(Player2Color).Count > 0;
	Boolean CanPlayer1MakeMove = board.generateMoves(Player1Color).Count > 0;


	Board.PlayerColor PlayerColorForNextTurn = this.Player2Color;
	if (!CanPlayer2MakeMove)
	{
		if (CanPlayer1MakeMove)
		{
		        //Passes
			PlayerColorForNextTurn = this.Player1Color;
		}
	}

	Board boardForHistory = (Board)this.board.Clone();
	boardForHistory.CurrentTurn = PlayerColorForNextTurn;
	AddCurrentBoardToHistory(boardForHistory, pos, this.Player1Color);
	UIBoard.RenderBoard();

	if (!CanPlayer2MakeMove)
	{
		if (!CanPlayer1MakeMove)
		{
		        //Both Player cannot make move
			this.GameState = GameStatusEnum.Finished;
			CalculateResult();

			UIBoard.InformGameResult(this.GameResult);
		}
		else
		{

			if (!IsPlayer2Bot || !IsPlayer1Bot)
			{
				this.UIBoard.InformPlayer2NeedtoPass();
			}
			board.SwitchTurnDueToPlayerPass();
		}
	}
}

protected virtual void OnPlayer1Moved()
{

	UIBoard?.RenderHistory();
	NextTurn();

}

private void NextTurn()
{
	if (this.GameState != GameStatusEnum.Playing)
	{
		return;
	}
	if (this.IsPlayer1NeedtoPass() && this.IsPlayer2NeedtoPass())
	{
		this.GameState = GameStatusEnum.Finished;
		CalculateResult();
		this.UIBoard.InformGameResult(this.GameResult);
		return;
	}
	bool CanMove = false;
	if (this.board.CurrentTurn == Board.PlayerColor.Black)
	{
		if (this.IsPlayer1Bot)
		{

			BotPlayer1MoveDecision(out CanMove);

			return;
		}
		else
		{
			if (this.IsPlayer1NeedtoPass())
			{
				this.UIBoard.InformPlayer1NeedtoPass();
				this.board.SwitchTurnDueToPlayerPass();
				this.UIBoard.RenderBoard();
			}
		}
		return;
	}

	if (this.IsPlayer2Bot)
	{

		BotPlayer2MoveDecision(out CanMove);
		return;
	}
	else
	{
		if (this.IsPlayer2NeedtoPass())
		{
			this.UIBoard.InformPlayer2NeedtoPass();
			this.board.SwitchTurnDueToPlayerPass();
			this.UIBoard.RenderBoard();
		}
	}

}

```
  
![Seq_Diagram](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/SEQ_Diagram_WhenUSerClick.PNG)   
This diagram is a simplified version of a sequence diagram.
It is just a simplified because.
1. Click(), CellClick() UIBoard_CellClick() are events that cannot be displayed on the sequence diagram easily.
2. The pictureBoxBoard is not an actual object that listens to Click() event, it is another picturebox that pictureBoxBoard contains.


### BoardHistory.cs
  This is a class that keeps track of the position each time the disk is put. 
  It also has the methods to navigate through these histories.  
### GameBuilder.cs
  Since this game is quite complicated so we need the GameBuilder object  
  to create a game object instead of using a constructor.  
  This is an example of how to create a game.  
```C#
		game = GameBuilder.Builder.BeginBuild
                     .GameMode(KReversiGame.PlayerMode.FirstHuman_SecondBot)
                     .BotPlayer1Is(BotPlayer1) // BotPlayer1 can be null.
                     .BotPlayer2Is(BotPlayer2) // BOtPlayer2 can be null.
                     .Player1NameIs(Global.Player1Name)
                     .Player2NameIs(Global.Player2Name)
                     .AllowRandomDecision (Global.CurrentSettings.IsAllowRandomDecision) 
                     .UsingAlphaBeta (Global.CurrentSettings.IsUsingAlphaBeta) 
                     .KeepLastDecisionTree (Global.CurrentSettings.IsKeepLastDecisionTree) // In case we need to view the minimax tree
                     .OpenWithBoardFile(CurrentBoardFileName)  //Path of .brd file, it can be blank in case of a new game.
                     .OpenWithGameFile(CurrentGameFileName) //Path of .rvi file, it can be blank in case of a new game.
                     .FinishBuild();
```
### Utility.SerializeUtility.cs
  This is the class we use to Serialize and Deserialize any kind of file/object.  
  
### Utility.UI.cs

  It has methods related to making a Dark mode.  
Explain the important code  
  These are the important codes that do not relate to AI.  
	
  **AI.Board.IsLegalMove()**
  1. If the row and column are not in the value range return false;  
  2. If Cell is not a blank cell return false;
  3. When we tried to loop for 8 directions to check the next cell, 
  we don't need to have 8 duplicate sets of code.  
  We can just use the loop for row and col.  
  like this    
  They get 9 unique results from this nested loop.  
  We only need 8 of them for the row and column change 
  in each direction.
  
  Loop Row from -1 to 1
    Loop Col from -1 to 1
     
     Row -1, Col -1 means North West.  
     Row -1, Col 0 means North.  
     Row -1, Col 1 means North East.  
     Row  0, Col -1 means West.  
     Row  0, Col 0 mean there is no change, just skip
     Row  0, Col 1 means East.  
     Row  1, Col 1 means South East.  
     Row  1, Col 0 means South.  
     Row  1, Col -1 means South West.  
     

![Direction](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/Direction.png)   
  
 In this picture the middle cell represents a cell we need to check.  
 It is in row 0, column 0.  
     We know that North West cell: Row is -1, Col is -1, we get it from 0 -1, 0 -1    
     We know that North cell     : Row is -1, Col is 1, we get it from 0 -1, 0 -0    
and so on.
     
     
     
	
For the condition, the cell must have at least one opponent cell first.  
Then it must have the same side cell.  

![UIControl04](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/UIControl04.png)  

In this picture only D1 is a valid position for a black turn.

```C#
public Boolean IsLegalMove(Position Position,  PlayerColor cValue)
{
    if (Position.Row < 0 ||
        Position.Col < 0 ||
        Position.Row > this.boardMatrix.GetLength (0)-1 ||
        Position.Col > this.boardMatrix.GetLength(1) - 1)
        {
	     //Row and Col must be in range.
             return false;
        }
        //Not empty cell
        if (GetCell(Position) != CellValue.Blank)
        {
            return false;
        }
        CellValue OponentValue = GetCellValueFromPlayerColor(GetOponentValue(cValue));
        int iRowChange;
        int iColChange;
        Position PositionCheck = Position;

        for (iRowChange = -1; iRowChange <= 1; iRowChange++)
        {
            for (iColChange = -1; iColChange <= 1; iColChange++)
            {
                if (iRowChange == 0 && iColChange == 0)
                {
		        // just skip its self cell.
                        continue;
                }
                Boolean HasAtLeasOneOpoonentColor = false;
                PositionCheck = Position.Clone();
                while (true)
                {
                    PositionCheck.Row += iRowChange;
                    PositionCheck.Col += iColChange;

                    if (PositionCheck.Row < 0 ||
                        PositionCheck.Row > 7 ||
                        PositionCheck.Col < 0 ||
                        PositionCheck.Col > 7)
                    {
                        break;
                    }
                       
                    if (CellsByPostion(PositionCheck) == CellValue.Blank)
                    {
                        break;
                    }
                    if (CellsByPostion(PositionCheck) == GetCellValueFromPlayerColor(cValue))
                    {
                        if (!HasAtLeasOneOpoonentColor)
                        {
			    //Encouter the same color without meeting the Opponent
			    //Case D5
                            break;
                        }
                        else
                        {
			    //If it can reach this poit it means it already hasAtLeastOne Opponenet
			    //Then it meet the same color
			    //Case D1
                            return true;
                        }

                    }
                    if (CellsByPostion(PositionCheck) == OponentValue)
                    {
                        HasAtLeasOneOpoonentColor = true;
                    }
                }
            }
        }
    return false;
}
```
**PutAndAlsoSwithCurrentTurn()**

  The logic behind this method is similar to **IsLegalMove()** method. 
  After it called **SetCell()** method to put the cell value then  
  It needs to loop in all 8 directions, when It found the position it can flip,  
  it will keep in the PositionFlip object.  
  After it has finished checking for 8 directions, it will call SetCell methods to  
  flip the cell.
		
# Introduction to AI Part
## How does MiniMax work
Before I would like to talk about Minimax,  
I would like to introduce you to a decision tree first.  
The image below depicts an example of a decision node tree for the Tic Tac Toe game.  

1. There is a blank state.
2. There are nine possible positions O can put.
3. There are eight positions that can be put for each of the positions that O puts in #2.
4. There are seven positions that can be put for each of the positions that X puts in #3.   

At#3 there are totally  
	81 nodes from  9 + (8 * 9)  
	72 possibilities from 9 * 8.  
At#4 there are totally  
	585 nodes from 9 + (8 * 9) + (72 * 7)  
	504 possibilities from 9 * 8 * 7.  


In total, there are 362,880 possibilities to fill the board with O and X in the 3x3 board size.  
This value can be calculated as x = 9 * 8 * 7 * 6 * 5 * 4 * 3 * 2.  
However, the number of possible moves in a game of tic-tac-toe will be less than this  
because some games will end before the board is full.  


![Node Tree](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/S/S_Nodes_Tree.png)   

If there are not too many nodes, we just search every node in the tree   
till we achieve our end goal.  
However, the Reversi game has a tree with roughly 10 ^ 28 nodes.  
We utilize Minimax to search for a limit-specific level with the node  
that is likely to be relevant because it is not practicable to calculate all of them.  


There are 2 parts to the Minimax algorithm  
**1. Evaluate function**
It is  the function that accepts board value as a parameter and then returns the score.  
We use this function because we would like to know if our position is good or bad.

```C#
// This is the concept
int score = Evaluate(board);
```

 **2. Search Tree**
Is a recursive tree to find the optimized value.

This is an example of using Minimax minimax algorithm on a tree.  

Supposing you need to decide to choose the best weapon to attack your enemy, while your enemy will   
decide to choose the best shield to protect themselves. 
Which weapon you will choose ?  

![Node Tree](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/MiniMaxAttack.png)   

You are supposed to choose the Maximum value
Your enemy is supposed to choose the Minimum value
and this is a Minimax.  

![BinaryTreeMinimax](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/BinaryTreeMinimax.png)     


The evaluation function will happen only in these 6 nodes  
LShield1, LShield2, FShield1, FShield2, IShield1, IShield2 

These are the order of progess.  
The value after the : is the value from processing.  
1. LShield1:100  
2. LShield2:200  
3. Lighting = Find Min(LShield1, LShield2) :100  
4. FShield1:110  
5. FShield2:90  
6. Fire= Find Min(FShield1, FShield2) :90  
7. IShield1:120  
8. IShield2:150  
9. ICE= Find Min(IShield1, IShield2) : 120  
10. Which Magic to Attack = Find Max(Lighting, Fire, ICE) :120  

The reason why I explain the order of the node to be executed is  
when the first time I learn Minimax.  
I mistakenly that this is the order being processed.  
1. Lighting:100  
2. Fire:90  
3. ICE:120  
4. LShield1:100  
5. LSheidl2:200  
...  

I was mistaken because the tree image shows the value 
after every node was calculated.
So I don't know what happened first.

The Lighting, Fire, ICE nodes were never be evaluated.  
They just use the value from their children nodes,  
only the leaf node will be evaluated.  
 

## Alpha Beta Pruning
Please look at the tree

(I adjusted the value of FShiled1 to 90)

![BinaryTreeMinimax](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/BinaryTree_AlphaBeta01.png)   

According to this tree, you no need to evaluate for FSheild2  
The reason that you can eliminate it due to the fact that

Lighting = MIn(100,200)  
Fire = Min(90,?)  
Result = Max(Lighting,Fire)  

We know that the result will be 100 regardless of the value of FShield2 node.

How do we know?

Let's try to substitute ? with 50 and check the result.

Result = Max(Min(100,200), Min(90,50))  
Result = Max(100,50)  
Result = 100  

Try again, this time we use 300  
Result = Max(Min(100,200), Min(90,300))  
Result = Max(100,90)  
Result = 100  

You can try to change the value in ? and finally you will get 100 as a result.  

You see that when we know these 2 things
1. The Fire node value is 100  
2. The Minimum value of the Lighting node is less than 90  

We can eliminate the Lighining at all.  
The Alpha-beta pruning is used to eliminate the number of nodes when we minimax algorithm.
You just use it, but in this example, we only have Beta 
because our tree is not deep enough, so it only has Min level node.

Alpha stores the best value that Maximizer can guarantee. 
	Its initial value is -Infinity. 
	We can just use int.MinValue instead of -Infinity.  
Beta stores the best value that Minimizer can guarantee.  
	Its initial value is Infinity.  
	We can just use int.MaxValue instead of Infinity. 

With Alpha Beta pruning,  
1. LShield1:100
2. LShield2:200 
3. Lighting = Find Min(LShield1, LShield2) :100  Set Beta to 100  
4. FShield1:90 Since The value is 90 it is less than Beta, we can ignore the rest of  
 the children  of the Fire node.  
 
This is a concept of how we use alpha and beta values in Minimax function.
Alpha is for the Maximize player to update.
Beta is for the Minimize player to update.
You can look into this it will help you more understand.
https://bsmith156.github.io/Alpha-Beta-Visualizer/

```C#
//This is a concept, not actual function
void Minimax(Node node,
			 int depth,
			 boolean isMaximize, 
			 int alpha, 
			 int beta)
{
	if (depth==0)
	{
		return Evaluation(node.board);
	}
	int bestValue = int.Maxvalue;
	if(isMaximize){
		bestValue = int.Minvalue;
	}
	foreach(Board childboard in node.board.GenerateMoves)
	{
		 Node childnode =new Node(childboard);
		if(isMaximize)
		{
		       ;
			int Score = Minimax(childnode, depth-1,!isMaximize,alpha,beta);
			bestValue = Math.Max(bestValue, Score); 
			alpha = Math.Max(alpha, bestValue);
			if(alpha > beta)
			{
				break;
			}
		
		} 
		else 
		{
			int Score = Minimax(childnode, depth-1,!isMaximize,alpha,beta);
			bestValue = Math.Min(bestValue, value);
			beta= Math.Min(beta, bestValue);
			if(alpha > beta)
			{
				break;
			}
		}
	}
	return bestValue
}
```


# The Important Classes that relate to AI

## BasicMiniMaxEvaluate Calculate

How does our evaluation function works?  

First, I would like to introduce you to the concept of PositionScore.   
In the Reversi game, the corner is considered a good position because it cannot be flipped by an opponent.  
Since the corner is good, the cell next to the corner is bad because it allows your opponent to occupy the corner.  

![XSquare](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/S/S_009_XSquare.png)    
In this picture every cell black can put the disk is a bad position   
because after black put the disk in these positions,  
the white will be able to get to the corner immediately.   

![Position Score](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/013_Evaluate_01_PositionScore.png)    
We can set the position score like this. 
The corner is good so we give it 100, while the next to the corner is bad we set them 
to -50  and -100 while for the other cell we set it to 1  

This evaluation function is strong enough to defeat a relatively inexperienced player, 
but it has no chance of competing against a Reversi expert.

The problem I found about this PostionScore is it will not choose a cell next to the border despite 
it is a good position.  
![Position Score](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/S/S_0011_NotBad_XSquare.png)

This picture, it shows that cell G2 is in a good position for black because it will allow him to have a 
corner at A8.  
Not all of the next-to-border cell is bad, sometimes it is good but the program cannot understand.


So I tried to look at a better method to improve AI and I found this article  
https://www.codeproject.com/Articles/24461/Reversi  
from  Zdravko Krustev  

These 2 functions I follow the same logic from him.

**GetNumberofStableDiscsFromCorner()**  
**GetStableDiscsFromFullEdge()**


After I implement these 2 evaluation  variables, the AI improved a lot.


**Calculate()**  
  **IsEndGame** if the game is ended, this function will just determine the result: win, loss, or draw.  
  **IsUseStatbleDiskScore** if this value is true it will call GetNoStableDisk method  
  **IsUseScoreFromPosition**	
  In the end, it will compare BotScore with OpponentScore and then return ResultScore.    
This method will do evaluation function, it will calculate the score for both sides (Botscore and Opponentscore)
  Then it returns BotScore - OpponentScore.  
  
**GetNoofDiskCanBeFlipped()**	
  loop through all of the possible moves    
  Try to put the disk  
  check how many disks can be flipped then keep then add the value    
**GetNoStableDisk()**  
  This is a method to check all 4 corners of the board and all 4 Edges of the board.
  
  
**GetNumberofStableDiscsFromCorner()**  
  Then call check all of the full edges on the 4 sides of the board.    
![Stable Disc from Corner](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/S/S_013_Evaluate_03_Corner_Is6.png)    
Stable disk from Corner is that  we count the number of the stable from each corner.  
In this picture, we count that there are 5 stables disks, we don't count the disk on C2 because it does not follow the pattern.  

**GetStableDiscsFromFullEdge()**  
![Stable Disc from Full Edge](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/S/S_013_Evaluate_05_Edge_6.png)  
The stable disk from Full edge is we count that in case of the edge is full we count how many of our disk between the opponent.  
In this picture is it 6.

```C#

public int getScore(Board board, bool IsMyTurn, Board.PlayerColor BotColor)
{
	var boardphase = board.BoardPhase;
	EvaluateScore BotScore = new EvaluateScore();
	EvaluateScore OpponentScore = new EvaluateScore();

	bool IsUseStatbleDiskScore = this.StableDiskScore != 0;
	bool IsUseAvailableScore = this.ScoreNumberMove(boardphase) != 0;
	// bool IsUseScoreFromPosition = true;

	LogDebugL2("IsUseStatbleDiskScore::" + IsUseStatbleDiskScore);
	LogDebugL2("IsUseAvailableScore::" + IsUseAvailableScore);
	LogDebugL2("IsUseScoreFromPosition::" + IsUseScoreFromPosition);

	int[,] positionScore = PositionScore(boardphase);

	return Calculate(board, IsMyTurn, BotColor,
		BotScore,
		OpponentScore,
		positionScore,
		this.StableDiskScore,
		this.ScoreNumberMove(boardphase),
		IsUseScoreFromPosition);

}

public int Calculate(Board board, bool IsMyTurn, Board.PlayerColor BotColor,
	EvaluateScore BotScore,
	EvaluateScore OpponentScore,
	int[,] ppositionScore,
	int StableDiskScore,
	int MobilityPieceScore,
	Boolean IsUseScoreFromPosition)
{

	bool IsUseStatbleDiskScore = StableDiskScore != 0;

	LogDebugL2("IsUseStatbleDiskScore::" + IsUseStatbleDiskScore);
	LogDebugL2("IsUseScoreFromPosition::" + IsUseScoreFromPosition);

	Board.PlayerColor OppoColor = board.GetOponentValue(BotColor); // ((Board)board).OpponentTurn;
	List<Position> listNoofPossibleMoveForBot = board.generateMoves(BotColor);
	List<Position> listNoofPossibleMoveForOpponent = board.generateMoves(OppoColor);
	BotScore.NoofPossibleMove = listNoofPossibleMoveForBot.Count;
	OpponentScore.NoofPossibleMove = listNoofPossibleMoveForOpponent.Count;

	Boolean IsEndGame = false;
	Boolean IsIWon = false;
	Boolean IsDraw = false;
        // There is no moveable position from both player
	if (BotScore.NoofPossibleMove == 0 &&
		OpponentScore.NoofPossibleMove == 0)
	{
		IsEndGame = true;
	}
       
	if (IsUseScoreFromPosition)
	{
		GetScoreFromPosition((Board)board, ppositionScore, BotColor, BotScore,
			OpponentScore);
	}
	if (IsEndGame)
	{
		LogDebugL2("IsEndGame is true");
		//In case the game is end
		//We just check the disk of both player
		//To decide to result
		BotScore.DiskCount = board.NumberofDisk(BotColor);
		OpponentScore.DiskCount = board.NumberofDisk(OppoColor);
		if (BotScore.DiskCount == OpponentScore.DiskCount)
		{
			IsDraw = true;
			LogDebugL2("IsDraw is true");
		}
		else
		{
			IsIWon = BotScore.DiskCount > OpponentScore.DiskCount;
			LogDebugL2("IsIWon is true");
		}

		if (IsDraw)
		{
			return 0;
		}
		if (IsIWon)
		{
			return WonScore;
		}

		return LostScore;
	}

	Boolean IsINeedtoPass = false;
	Boolean IsEnemyNeedtoPass = false;
	if (OpponentScore.NoofPossibleMove == 0 &&
		BotScore.NoofPossibleMove > 0)
	{
		IsEnemyNeedtoPass = true;
	}

	if (BotScore.NoofPossibleMove == 0 &&
		OpponentScore.NoofPossibleMove > 0)
	{
		IsINeedtoPass = true;
	}

	if (IsEnemyNeedtoPass)
	{
		BotScore.ScorePassWeight += this.ForcePassSocre; //Get score when we can forece to pass
	}
	if (IsINeedtoPass)
	{
		OpponentScore.ScorePassWeight += this.ForcePassSocre; //Opponent get score when it can forece us to pass
	}

	if (IsUseStatbleDiskScore)
	{

		BotScore.NoofStableDisk = GetNoStableDisk(board, BotColor);
		OpponentScore.NoofStableDisk = GetNoStableDisk(board, OppoColor);
		BotScore.ScoreStableDiskWeight = BotScore.NoofStableDisk * StableDiskScore;
		OpponentScore.ScoreStableDiskWeight = OpponentScore.NoofStableDisk * StableDiskScore;
	}


	BotScore.NoofDiskCanFlip = GetNoofDiskCanBeFlipped(board, BotColor, listNoofPossibleMoveForBot);
	OpponentScore.NoofDiskCanFlip = GetNoofDiskCanBeFlipped(board, OppoColor, listNoofPossibleMoveForOpponent);
	BotScore.ScoreDiskCanFlip = MobilityPieceScore * BotScore.NoofDiskCanFlip;
	OpponentScore.ScoreDiskCanFlip = MobilityPieceScore * OpponentScore.NoofDiskCanFlip;

	LogDebugL2("===BotScore");
	LogDebugL2("BotScore.NoofPossbleMove::" + BotScore.NoofPossibleMove);
	LogDebugL2("BotScore.NoofStableDisk::" + BotScore.NoofStableDisk);

	LogDebugL2("BotScore.ScoreStableDiskWeight::" + BotScore.ScoreStableDiskWeight);
	LogDebugL2("BotScore.ScorePositionWeight::" + BotScore.ScorePositionWeight);
	LogDebugL2("BotScore.ScorePassWeight::" + BotScore.ScorePassWeight);
	LogDebugL2("BotScore.ScoreDiskCanFlip::" + BotScore.ScoreDiskCanFlip);

	LogDebugL2("===OpponentScore");
	LogDebugL2("OpponentScore.NoofPossbleMove::" + OpponentScore.NoofPossibleMove);
	LogDebugL2("OpponentScore.NoofStableDisk::" + OpponentScore.NoofStableDisk);

	LogDebugL2("OpponentScore.ScoreStableDiskWeight::" + OpponentScore.ScoreStableDiskWeight);
	LogDebugL2("OpponentScore.ScorePositionWeight::" + OpponentScore.ScorePositionWeight);
	LogDebugL2("OpponentScore.ScorePassWeight::" + OpponentScore.ScorePassWeight);
	LogDebugL2("OpponentScore.ScoreDiskCanFlip::" + OpponentScore.ScoreDiskCanFlip);
	EvaluateScore ResultScore = BotScore - OpponentScore;

	int ScoreTotal = ResultScore.GetTotalScore();

	return ScoreTotal;
}


public int GetNoStableDisk(Board board, Board.PlayerColor color)
{
    int NofromCorner =
		this.GetNumberofStableDiscsFromCorner(board, color, Corner.TopLeft) +
		this.GetNumberofStableDiscsFromCorner(board, color, Corner.TopRight) +
		this.GetNumberofStableDiscsFromCorner(board, color, Corner.BottomLeft) +
		this.GetNumberofStableDiscsFromCorner(board, color, Corner.BottomRigth);

    int NofromEdge =
		this.GetStableDiscsFromFullEdge(board, color, NEWS.North) +
		this.GetStableDiscsFromFullEdge(board, color, NEWS.East) +
		this.GetStableDiscsFromFullEdge(board, color, NEWS.West) +
		this.GetStableDiscsFromFullEdge(board, color, NEWS.South);

    return NofromCorner + NofromCorner;

}
		
public int GetNumberofStableDiscsFromCorner(Board board, Board.PlayerColor color, Position PositionCorner)
{
    int noofDisk = 0;
    int rowDelta = 1;
    int columnDelta = 1;
    if (PositionCorner.Row != 0)
    {
		rowDelta = -1;
    }
    if (PositionCorner.Col != 0)
    {
		columnDelta = -1;
    }

    int LastRow = 7;
    int LastColumn = 7;
    if (PositionCorner.Row != 0)
    {
		LastRow = 0;
    }
    if (PositionCorner.Col != 0)
    {
		LastColumn = 0;
    }
    for (int indexRow = PositionCorner.Row; indexRow != LastRow; indexRow += rowDelta)
    {
		int indexColumn = 0;
		for (indexColumn = PositionCorner.Col; indexColumn != LastColumn; indexColumn += columnDelta)
		{
			if (board.boardMatrix[indexRow, indexColumn] != (int)color)
			{
				break;
			}
			noofDisk++;
		}
		Boolean IsThereColumnNeedtoCheck = false;
		IsThereColumnNeedtoCheck =
			(columnDelta > 0 && indexColumn < 7) ||
			(columnDelta < 0 && indexColumn > 0);
		if (!IsThereColumnNeedtoCheck)
		{
			continue;
		}

		LastColumn = indexColumn - columnDelta;
		if (columnDelta > 0 && LastColumn == 0)
		{
			LastColumn++;
		}
		else if (columnDelta < 0 && LastColumn == 7)
		{
			LastColumn--;
		}

		if ((columnDelta > 0 && LastColumn < 0)
		|| (columnDelta < 0 && LastColumn > 7))
		{
			break;
		}
    }
    return noofDisk;
}

public int GetStableDiscsFromFullEdge(Board board, Board.PlayerColor color, NEWS news)
{
    Board.PlayerColor OppositeColor = Board.PlayerColor.Black;
    if (color == Board.PlayerColor.Black)
    {
		OppositeColor = Board.PlayerColor.White;

    }
    if (!IsEdgeFull(board, news))
    {
		return 0;
    }
    int result = 0;
    Position positionBegin = null;
    Position positionEnd = null;
    GetPostionBeginandEnd(news, ref positionBegin, ref positionEnd);

    bool hasFoundOppositeColor = false;
    int NoofRepeatedDisk = 0;
    for (int iRow = positionBegin.Row; iRow <= positionEnd.Row; iRow++)
    {
		for (int iColumn = positionBegin.Col; iColumn <= positionEnd.Col; iColumn++)
		{
		Board.PlayerColor DiskColor = (Board.PlayerColor)board.boardMatrix[iRow, iColumn];
			if (!hasFoundOppositeColor &&
				DiskColor == OppositeColor)
			{
				hasFoundOppositeColor = true;
				NoofRepeatedDisk = 0;
				continue;
			}
			if (hasFoundOppositeColor)
			{
				if (DiskColor == color)
				{
					NoofRepeatedDisk++;
				}
				else
				{
					result += NoofRepeatedDisk;
					NoofRepeatedDisk = 0;
				}
			}

		}
    }
    return result;
}
```

![014 BotConfigure](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/014_BotConfigure.png)  
How to configure a Bot  
You can create or update the bot.

The non-tabs area.   
1. Choose a photo: just click to choose a photo  
2. Bot Name: The name of the bot  
3. Time limit per move(seconds): Supposing the value is 5, the bot is allowed to use a maximum of 5 seconds per turn.  
4. Stable disk score: This value will be multiplied by the number of stable disks.  
	Supposing this value is 20  
	The number of stable disks is 10  
	The Total Stable disk score will be 200 = 20 * 10  
	
5. Force pass score: In a Reversi game, when your opponent does not have  
a valid move, the game will switch to your turn.  
	If the position of the board forces your opponent to pass, you can get this score.   
6. Check Use Position Score: If it is not checked, the bot will not use Position Score.  

The tabs area.     

We have 3 tabs, and each tab represents the value the bot will use during the game  
We allow configuring the value for Openning, Middle, End Game phases.  
N = Number of white disk + Number of black disk;  
Begining : N <= 20  
Middle: N <=40  
End game: N > 40  

1. Score Position: You see that there is an array of textbox, you can configure the score position.  
2. Score Mobilize: The number of available moves multiplied by Score Mobilize.  
	if this value is high, the bot will try to get an available move as much as it can.  
3. Depth Level: You can configure the depth level, I suggest that it should not be more than 7  
so the bot will not take too long to calculate. If you configure this value more than 7, please  
make sure that you have configured the Time limit per move.  
4. Copy cell to Middle, Copy cell to End Game: In case you would like to copy the Score Position  
to another tab, just click the button.  

## UI\PanelPositionBotConfigure.cs
  This class inherits from Panel, it contains an array of 64 Textboxes to let the user   
  enter the score value for evaluating function.  
  It allows you use only need to configure the score on A1 to D4 only then the other 3 parts of the board   
  will change the value to match your A1 to D4 value.
  
## UI\PanelBotConfigure.cs
  This class inherits from Panel, it consists of a tab control that  
  has 3 tab pages, Opening, Middle, and End Game.  
  Each page contains PanelPositionBotConfigure, Score Mobilize, and Depth Level.

## AI\MiniMaxBotProto.cs
This class is a class that implements IPlayer interface, so we just need to implement  
MakeMove(AI.IBoard pBoard) method.  

The value you configured when you create a bot, will be stored on a file then it will 
be deserialized then fill into the object from this class.

The main purpose of this class is to just store the evaluation function parameter and then   
call the Minimax class to execute it, there is no calculation in this class.  

```C#
[Serializable]
public class MiniMaxBotProto : IPlayer
{
	//private IEvaluate Evaluate;
	private BasicMiniMaxEvaluate Evaluate = null;
	public MiniMaxBotProto()
	{

	}
	/*
	 These value will be set to Evaluate object.
	 The main purpose of this class is to hold these value
	 */
	public int ForcePassSocre { get; set; } = 1000;
	public int StableDiskScore { get; set; } = 10;
	public int TimeLimitPermove { get; set; } = 3;// Number of second bot allowed to think in each turn.
	// public Boolean AllowRandom { get; set; } = false;

	public int ScoreNumberMoveAtBegingGame { get; set; }
	public int ScoreNumberMoveAtMiddleGame { get; set; }
	public int ScoreNumberMoveAtEndGame { get; set; }


	public int DepthLevelAtBeginGame { get; set; }
	public int DepthLevelAtMiddleGame { get; set; }
	public int DepthLevelAtEndGame { get; set; }
	
	public string BotName { get; set; }

	public string Base64Image { get; set; }
	// public string PhotoFileName { get; set; }
	// As of now don't use FileName, use Base64Image instead.



	/*
	 These are default value of PostionScore
	 For Open, Mid, End game.
	*/
	public int[,] OpenGamePostionScore = new int[,] {
		{ 100, -50, 20, 5, 5, 20, -50, 100},
		{-50 , -70, -5, -5, -5, -5, -70, -50},
		{20  , -5 , 15, 3, 3, 15, -5, 20},
		{5   , -5 , 3, 3, 3, 3, -5, 5},
		{5   , -5 , 3, 3, 3, 3, -5, 5},
		{20  , -5 , 15, 3, 3, 15, -5, 20},
		{-50 , -70, -5, -5, -5, -5, -70, -50},
		{100 , -50, 20, 5, 5, 20, -50, 100}
	};


	public int[,] MidGamePostionScore = new int[,] {
		{ 140, -20, 20, 5, 5, 20, -20, 140},
		{-20 , -40, -5, -5, -5, -5, -40, -20},
		{20  , -5 , 15, 3, 3, 15, -5, 20},
		{5   , -5 , 3, 3, 3, 3, -5, 5},
		{5   , -5 , 3, 3, 3, 3, -5, 5},
		{20  , -5 , 15, 3, 3, 15, -5, 20},
		{-20 , -40, -5, -5, -5, -5, -40, -20},
		{140 , -20, 20, 5, 5, 20, -20, 140}
	};

	public int[,] EndGamePostionScore = new int[,] {
		{ 20, -5, 10, 5, 5, 10, -5, 20},
		{-5 , -10, 5, 5, 5, 5, -10, -5},
		{20  , 5 , 5, 5, 5, 5, 5, 10},
		{5   , 5 , 5, 5, 5, 5, 5, 5},
		{5   , 5 , 3, 5, 5, 5, 5, 5},
		{10  , 5 , 5, 5, 5, 5, 5, 10},
		{-5 , -10, 5, 5, 5, 5, -10, -5},
		{20 , -5, 10, 5, 5, 10, -5, 20}
	};

	public int DepthLevel(Board.BoardPhaseEnum boardPhase)
	{
		//Select depth level according to the BoardPhase
		int depthLevel = 0;
		switch (boardPhase)
		{
			case Board.BoardPhaseEnum.Begining:
				depthLevel = DepthLevelAtBeginGame;
				break;
			case Board.BoardPhaseEnum.Middle:
				depthLevel = DepthLevelAtMiddleGame;
				break;
			case Board.BoardPhaseEnum.EndGame:
				depthLevel = DepthLevelAtEndGame;
				break;
			default:
				depthLevel = 1;
				break;
		}
		if (depthLevel < 1 ||
				depthLevel > 10)
		{
			throw new Exception("Depth level is invalid");
		}

		return depthLevel;
	}
	public void FillEvaluateObject()
	{
		// this.Evaluate.ScoreNumberMoveAtBegingGame = this.ScoreNumberMoveAtBegingGame;

		this.Evaluate = new BasicMiniMaxEvaluate(this);
	}

	public bool IsUseScoreFromPosition = true;

	[NonSerialized()]
	public bool IsAllowRandomDecision = false;

	[NonSerialized()]
	public bool IsKeepLastDecisionTree = false;

	[NonSerialized()]
	public bool IsUsingAlphaBeta = false;

	[NonSerialized()] MiniMax m = null;
	public Position MakeMove(IBoard pBoard)
	{
	        // Using Minimax class to call calulcatNextMove() method
		Board.PlayerColor BotColor = ((Board)pBoard).CurrentTurn;
		int depthLevel = DepthLevel(((Board)pBoard).BoardPhase);

		Utility.TimeMeasure time = new Utility.TimeMeasure();
		time.Start();
		Boolean IsSortedNode = true;
		if (m == null)
		{
			m = new MiniMax();
		}
		Position result = m.calculateNextMove(pBoard,
			depthLevel,
			Evaluate,  // Evaluation object
			BotColor,
			this.TimeLimitPermove,
			IsUsingAlphaBeta, 
			IsKeepLastDecisionTree, // For viewing Minimax node later.
			IsAllowRandomDecision, // To prevent bot vs bot playing the same everytime.
			IsSortedNode // For better performance, sort the node at some depth level
			);
		time.Finish();
		String strTemp = time.TimeTakes.Seconds.ToString();
		return result;

	}
	public static MiniMaxBotProto CreateBot(String fileName)
	{

		MiniMaxBotProto Newbot = Utility.SerializeUtility.DeserializeMinimaxBot(fileName);
		Newbot.FillEvaluateObject();
		return Newbot;

	}
	public static MiniMaxBotProto CreateBot()
	{
		//These are default value
		//They will be used incase we don't load Botvalue then FillEvaluateObject
		//
		MiniMaxBotProto minimaxBorProto = new MiniMaxBotProto();
		minimaxBorProto.DepthLevelAtBeginGame = 2;
		minimaxBorProto.DepthLevelAtMiddleGame = 3;
		minimaxBorProto.DepthLevelAtEndGame = 5;

		minimaxBorProto.ScoreNumberMoveAtBegingGame = 10;
		minimaxBorProto.ScoreNumberMoveAtMiddleGame = 40;
		minimaxBorProto.ScoreNumberMoveAtEndGame = 20;

		minimaxBorProto.DepthLevelAtBeginGame = 2;
		minimaxBorProto.DepthLevelAtMiddleGame = 2;
		minimaxBorProto.DepthLevelAtEndGame = 2;

		//IEvaluate Evu = new BasicMiniMaxEvaluate();
		minimaxBorProto.Evaluate = new BasicMiniMaxEvaluate(minimaxBorProto);
		return minimaxBorProto;
	}
}
```

## Keep the score to the cache.

![Duplicate Node](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/DuplicateResult.png)   
This picture shows an example of a duplicate node.  

## Hash.cs

When we create a search tree for the Minimax function, there is a chance  
that the tree node will have a duplicate value for another tree node.  
If that case happens, we don't want to do the evaluation again, so  
we decide to keep the score value in the hash object.  


Every time we call the evaluation function to calculate the score from the board  
We keep the score value in this Hash object, the next time when there is a duplicate node appears  
we just retrieve the value that we already stored.  
We also need to keep the depth level because the same board position has different 
scores for different levels.

```C#
public class Hash
{

	public static int ScoreForNonExist = int.MaxValue / 2;

	public static int GetHashForBoard(Board board)
	{
	        /* We need to use GetHashCode() multiple with the current turn
		because we need to know if it it black or white  turn.
		GetHashCode is just a function to calculate hash by simply
		multple with each member value.
		It can be improved to use XOR hash function.		
		*/
		return board.GetHashCode() * (int)board.CurrentTurn ;
	}
	
	/* For DicHash
		int is Hashvalue
		Dictionary<int,int>	is DicDepthScore
	   For DicCepthScore
	        First int is the depth level
		Second int is the score.
	
	*/
	private Dictionary<int, Dictionary<int, int>> DicHash = new Dictionary<int, Dictionary<int, int>>();
	private Dictionary<int, int> DicEvoScore = new Dictionary<int, int>();
	public int NumberofNodeCount()
	{
		int NumberofNodeCount = 0;
		foreach (int HashBoard in DicHash.Keys)
		{
			foreach (int DepthExist in DicHash[HashBoard].Keys)
			{
				NumberofNodeCount++;
			}
		}
		NumberofNodeCount += DicEvoScore.Count;
		return NumberofNodeCount;
	}
	/*
	Add new hash and evaluated score value
	We also need Depth parameter because the same board postion
	with different depth return the different value
	*/
	public void Add(int HashCodeForBoard, int Score, int Depth)
	{
		Dictionary<int, int> DicDepthScore = new Dictionary<int, int>();
		if (DicHash.ContainsKey(HashCodeForBoard))
		{
			//if this board value already exist
			//Search for the depth
			DicDepthScore = DicHash[HashCodeForBoard];
			if(DicDepthScore.ContainsKey(Depth))
			{
				return;
			}
			foreach(int DepthExist in DicDepthScore.Keys)
			{
				if(DepthExist >=Depth)
				{
					return;
				}
			}
			
		} else
		{
			DicHash.Add(HashCodeForBoard, DicDepthScore);
		}
		DicDepthScore.Add(Depth, Score);

	}
	public void AddEvalScore(int HashCodeForBoard, int Score)
	{
		// Dic
		if(DicEvoScore.ContainsKey(HashCodeForBoard))
		{
			return;
		}
		DicEvoScore.Add(HashCodeForBoard, Score);
	}
	public int GetEvalScore(int HashCodeForBoard)
	{
		if(DicEvoScore.ContainsKey(HashCodeForBoard))
		{
			return DicEvoScore[HashCodeForBoard];

		}
		return ScoreForNonExist;
	}


	public int GetScore(int HashCodeForBoard, int DepthLeast)
	{
		if (!DicHash.ContainsKey(HashCodeForBoard))
		{
			return ScoreForNonExist;
		}
		foreach (int DepthExist in DicHash[HashCodeForBoard].Keys)
		{
			if(DepthLeast <= DepthExist)
			{
				return DicHash[HashCodeForBoard][DepthExist];
			}
		}
		return ScoreForNonExist;
	}

	public Boolean ContainScore(int HashCodeForBoard, int DepthLeast)
	{
		return GetScore(HashCodeForBoard, DepthLeast) != ScoreForNonExist;
	}
	public Boolean ContainEvalScore(int HashCodeForBoard)
	{
		return GetEvalScore(HashCodeForBoard) != ScoreForNonExist;
	}
}
```    


### MinimaxParameterExtend.cs
We use this class to store the value of the minimax parameter.  
Actually, you don't need this class to use a minimax function,   
I created this class because I need to keep the List of MinimaxParameterExtend as a child because  
I need to show it later to the people who would like to learn how minimax works. 


```C#
[Serializable]
public class MiniMaxParameterExtend : MiniMaxParameter
{
	public List<MiniMaxParameterExtend> child = new List<MiniMaxParameterExtend>(); //We keep the children nodes here.
	public PositionScore PositionScore { get; set; } = new PositionScore();  // The selected position and its score.
	public MiniMaxParameterExtend CloneExtendWithoutBoard()
	{
		MiniMaxParameterExtend CloneObject = new MiniMaxParameterExtend();
		CloneObject.Depth = this.Depth;
		CloneObject.IsMax = this.IsMax;
		CloneObject.Alpha = this.Alpha;
		CloneObject.Beta = this.Beta;
		CloneObject.BotColor = this.BotColor;
		if (this.PositionScore != null)
		{
			CloneObject.PositionScore = this.PositionScore.ClonePositionScore();
		}
		return CloneObject;
	}
	public MiniMaxParameterExtend CloneExtend()
	{
		MiniMaxParameterExtend CloneObject = CloneExtendWithoutBoard();
		CloneObject.board = (Board)this.board.Clone();
		return CloneObject;
	}
} 
```    
### MiniMax.cs  
**calculateNextMove()**  
This method is just for preparing a parameter to call **MinimaxAlphaBetaExtend()**  

**MinimaxAlphaBetaExtend()**

1. if it is not the final move check need to pass  
   if the Bot color doesn't have a valid move switch to the opponent's color  
   then check if it has a valid move or not.  
   if both colors have no valid move, set IsFinalMove=true  
	
2. if it is the final move   
   check from the hash if this position is already calculated and store it in the hash or not  
   if it is get the Score from the hash  
   if it is not, call evaluate board function then store the score in a hash  
   return score  
	
3. It relates to sorting available moves.  
   the program will try to evaluate the score from the board and then try to sort the available moves.  
   We don't do this on every depth level.  
   The result from #3 will be sent to #4  
	
4. This is a part where we do the minimax.  
   loop through all of the Positions in the available move.  
   check if it can get a score from hash or not.  
   if it cannot just call MinimaxAlphaBetaExtend to get the childScore  
   store it in the hash object.  
   The result of this method is just ordinary minimax.  
```C#
public Position calculateNextMove(IBoard board,
	int depth,
	IEvaluate pEvaluateObject,
	Board.PlayerColor BotColor,
	int SecondsLimitPerMove,
	Boolean IsUsingAlphaBeta,
	Boolean IsKeepingChildValue,
	Boolean IsUsingRandomIfNodeValueIsTheSame,
	Boolean IsSortedNode)
{

	Log("CalculateNextMove begin");

	EvaluateObject = pEvaluateObject;

	Position move = new Position(-1, -1);

	PositionScore bestMove = new PositionScore();

        //This Para object will be sent to MinimaxAlphaBetaExtend()
	MiniMaxParameterExtend Para = new MiniMaxParameterExtend();

	Para.Depth = depth - 1;
	Para.board = (Board)board.Clone();
	Para.IsMax = true; // The initial node is for Maximum player
	Para.Alpha = int.MinValue; // Initial value of Alpha
	Para.Beta = int.MaxValue;  // Initial value of Beta
	Para.BotColor = BotColor;
	NodeCount = 0; 
	EvaluateCount = 0;
	EndTime = DateTime.Now.AddSeconds(SecondsLimitPerMove); // The end time that allow bot to use 
	Log("CalculateNextMove :: SecondLimitPerMove ::" + SecondsLimitPerMove);
	Log("CalculateNextMove :: Depth ::" + Para.Depth);


	Utility.TimeMeasure timeM = new Utility.TimeMeasure(); // To keep track how long does it takes
	iCountHashCanAccess = 0; //Keep track number of time Hash object can be access
	timeM.Start();
	this.FirstLevelDepth = Para.Depth;
	listFirstLevelDepthMoves = new List<PositionScore>();
	PositionScore score = MinimaxAlphaBetaExtend(Para,
		IsUsingAlphaBeta, 
		IsKeepingChildValue, // To show Minimax later
		IsSortedNode); // Improve searching by tryting to sort fist
	move = new Position(score.Row,
		score.Col);
	timeM.Finish();

	Log("CalculateNextMove :: [" + score.Row + "," + score.Col + "]");
	Log("CalculateNextMove :: Score::" + score.Score);
	Log("CalculateNextMove :: iCountHashCanAccess::" + iCountHashCanAccess);
	Log("CalculateNextMove ::  Before Get Random");
          
	if (Para.board.NumberofBothDisk() <= 14)
	{
	        // Random decision if Number of disk on board is <= 14
		// This feature exists to prevent bot vs bot select the same positino everytime
		if (IsUsingRandomIfNodeValueIsTheSame)
		{
			Log("CalculateNextMove ::  IsUsingRandomIfNodeValueIsTheSame");
			score = GetRandomFromMaxScoreNode(listFirstLevelDepthMoves);

			move = new Position(score.Row,
				score.Col);

		}
	}

	//NodeCount = 0;
	Log("CalculateNextMove :: hashTranportable :: " + hashTranportable.NumberofNodeCount());

	Log("CalculateNextMove :: [" + score.Row + "," + score.Col + "]");
	Log("CalculateNextMove :: Score::" + score.Score);
	Log("CalculateNextMove :: NodeCount ::" + NodeCount);
	Log("CalculateNextMove :: EvaluateCount ::" + EvaluateCount);
	Log("CalculateNextMove :: Time takes (Seconds)::" + timeM.TimeTakes.Milliseconds / 1000.00);
	Log("CalculateNextMove End");

	ClearMinimaxForDebug();
	_MiniMaxForDebug = Para;
	// Save MimimaxFor debug, to show later
	Utility.SerializeUtility.SerializeMiniMaxParameterExtend(_MiniMaxForDebug,
		Utility.FileUtility.MiniMaxParameterForDebugFilePath
		);

	return move;

}

private PositionScore MinimaxAlphaBetaExtend(
 MiniMaxParameterExtend Para,
 Boolean IsUsingAlphaBeta,
 Boolean IsKeepingChildValue,
 Boolean IsSortedNode
)
{

	String tab = Tab(Para.Depth);
	String methodName = tab + "Minimax::";
	NodeCount++;


	Board.PlayerColor DiskColor = Para.BotColor;
	Board.PlayerColor OpponentColor = ((Board)Para.board).GetOponentValue(Para.BotColor);

	LogDebug(methodName + "IsMax::" + Para.IsMax);
	if (!Para.IsMax)
	{
		DiskColor = OpponentColor;
	}
	Boolean IsMyTurn = false;
	if (DiskColor == Para.BotColor)
	{
		IsMyTurn = true;
	}
	if (IsExceedTimeLimitPermove())
	{
		LogDebug(methodName + " Exceed Time");
	}
	bool IsNeedtoPass = false;

	List<Position> avilableMovePositions = new List<Position>();
	bool isFinalMove = false;
	if (Para.Depth <= 0 || IsExceedTimeLimitPermove())
	{
		isFinalMove = true;
	}

	LogDebug(methodName + "IsFinalMove::" + isFinalMove);
	if (!isFinalMove)
	{

		avilableMovePositions = Para.board.generateMoves();
		if (avilableMovePositions.Count == 0)
		{
			LogDebug(methodName + "possbileMove.Count of " + DiskColor + "is 0 ");
			IsNeedtoPass = true;
			List<Position> avilableMovePositionsForOppositeColor = null;
			if (DiskColor == Para.BotColor)
			{
				avilableMovePositionsForOppositeColor = ((Board)Para.board).generateMoves(OpponentColor);
				DiskColor = OpponentColor;
				LogDebug(methodName + "Gen possbileMove from OpponentColor it is " + avilableMovePositions.Count);

			}
			else
			{
				avilableMovePositionsForOppositeColor = ((Board)Para.board).generateMoves(Para.BotColor);
				DiskColor = Para.BotColor;
				LogDebug(methodName + "Gen possbileMove from BotColor it is " + avilableMovePositions.Count);
			}

			avilableMovePositions = avilableMovePositionsForOppositeColor;
		}

		if (avilableMovePositions.Count == 0)
		{
			isFinalMove = true;
		}

	}

	PositionScore BestScore = new PositionScore(-1, -1);

	/* If it is final move
	   calculate the score then return
	*/
	if (isFinalMove)
	{
		int BoardHash = Hash.GetHashForBoard(Para.board);
		int Score = 0;
		if (hashTranportable.ContainEvalScore(BoardHash))
		{
			iCountHashCanAccess++;
			Score = hashTranportable.GetEvalScore(BoardHash);
		}
		else
		{

			EvaluateCount++;
			Score = this.EvaluateObject.evaluateBoard(Para.board, IsMyTurn, Para.BotColor);

			hashTranportable.AddEvalScore(BoardHash, Score);
			LogDebug("hashPutEvalScore::" + Para.board.LastPutPosition.PositionString());

		}

		LogDebug(methodName + "Score::" + Score);
		BestScore = new PositionScore(Score, -1, -1);
		return BestScore;

	}


	BestScore = new PositionScore(int.MaxValue, -1, -1);
	if (Para.IsMax)
	{
		BestScore.Score = int.MinValue;
	}
	// For Supporting sort node before doing Minimax
	// We don't sort the node at everydepth 
	if (IsSortedNode && Para.Depth >= this.FirstLevelDepth - 1)
	{
		LogDebug(methodName + "IsSortedNode is true");
		List<PositionScore> lstPostion = new List<PositionScore>();

		Dictionary<int, Board> DicBoard = new Dictionary<int, Board>();
		LogDebug(methodName + "Begin calculate score");
		foreach (Position nextMove in avilableMovePositions)
		{
			Board tempBoard = (Board)Para.board.Clone();

			tempBoard.PutAndAlsoSwithCurrentTurn(nextMove, DiskColor);
			EvaluateCount++;
			int Score = this.EvaluateObject.evaluateBoard(tempBoard, IsMyTurn, Para.BotColor);

			lstPostion.Add(new PositionScore(Score, nextMove.Row, nextMove.Col));
			DicBoard.Add(nextMove.GetHashCode(), tempBoard);

			LogDebug(methodName + "   Score::" + Score);
			LogDebug(methodName + "        move::" + nextMove.PositionString());
		}
		LogDebug(methodName + "End calculate score");
		List<PositionScore> SortedList = null;

		if (Para.IsMax)
		{
			SortedList = lstPostion.OrderByDescending(o => o.Score).ToList();
		}
		else
		{
			SortedList = lstPostion.OrderBy(o => o.Score).ToList();
		}

		LogDebug("Max Begin ViewSortedList");
		avilableMovePositions.Clear();
		foreach (PositionScore move in SortedList)
		{
			LogDebug(methodName + "   Score::" + move.Score);
			LogDebug(methodName + "        move::" + move.PositionString());

			avilableMovePositions.Add(new Position(move.Row, move.Col));
		}
		
	}
	// End For Supporting sort node before doing Minimax


	//Begin doing Minimax
	LogDebug(methodName + "Before loop");
	foreach (Position nextMove in avilableMovePositions)
	{

		LogDebug(methodName + "  position[" + nextMove.Row + "," + nextMove.Col + "]");
		MiniMaxParameterExtend childPara = Para.CloneExtend();


		LogDebug(methodName + "DiskColor::" + DiskColor);

		((Board)childPara.board).PutAndAlsoSwithCurrentTurn(nextMove, DiskColor);
		LogDebug(methodName + " After put");
		// For Viewing Minimax later
		if (IsKeepingChildValue)
		{
			Para.child.Add(childPara);
		}


		childPara.IsMax = !Para.IsMax;

		if (IsNeedtoPass)
		{

			childPara.IsMax = Para.IsMax;
			((Board)childPara.board).SwitchTurnDueToPlayerPass();
		}

		childPara.Depth--;

		int Score = 0;
		int BoardHash = Hash.GetHashForBoard(childPara.board);

		PositionScore childScore = null;
		int ScoreFromHash = hashTranportable.GetScore(BoardHash, childPara.Depth);
		if (ScoreFromHash != Hash.ScoreForNonExist)
		{
			iCountHashCanAccess++;
			childScore = new PositionScore(hashTranportable.GetScore(BoardHash, childPara.Depth));
		}
		else
		{
			childScore = this.MinimaxAlphaBetaExtend(childPara, IsUsingAlphaBeta, IsKeepingChildValue, IsSortedNode);
			hashTranportable.Add(BoardHash, childScore.Score, childPara.Depth);
			LogDebug("hashPut::" + nextMove.PositionString() + " depth::" + childPara.Depth + "::CUrrentTurn:: " + childPara.board.CurrentTurn + " :: BoardHash::" + BoardHash);
			//hashTranportable.Add(BoardHash, childScore.Score);
		}
		childPara.PositionScore = new PositionScore(childScore.Score, nextMove);

		if (Para.Depth == this.FirstLevelDepth)
		{
			LogDebug("First Level::" + nextMove.PositionString() + "   ::" + childScore.Score);
			listFirstLevelDepthMoves.Add(new PositionScore(childScore.Score, nextMove));

		}
		LogDebug(methodName + "  childScore.Score::" + childScore.Score);
		if (Para.IsMax)
		{
			LogDebug(methodName + "  BestScore.Score::" + BestScore.Score);
			if (childScore.Score > BestScore.Score)
			{
				LogDebug(methodName + "  childScore.Score  > BestScore.Score");
				BestScore = new PositionScore(childScore.Score, nextMove);
				if (IsUsingAlphaBeta)
				{
					Para.Alpha = Math.Max(BestScore.Score, Para.Alpha);

					if (Para.Beta < BestScore.Score)
					{
						LogDebug(methodName + "  BoardValue >= Beta");
						break;
					}
				}
			}
		}
		else
		{
			LogDebug(methodName + "  BestScore.Score::" + BestScore.Score);
			if (childScore.Score < BestScore.Score)
			{
				LogDebug(methodName + "  childScore.Score  < BestScore.Score");

				BestScore = new PositionScore(childScore.Score, nextMove);
				if (IsUsingAlphaBeta)
				{
					Para.Beta = Math.Min(Para.Beta, BestScore.Score);

					if (BestScore.Score <= Para.Alpha)
					{
						LogDebug(methodName + "  BoardValue <= Alpha");
						break;
					}
				}
			}
		}
	}
	LogDebug(methodName + "After Loop");
	return BestScore;

}



```

### Show Minimax Latest Move  
![Show Minimax](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/ShowMinimax.png)  
You can click on Game->Show MiniMax Latest Move menu.  
Please make sure that Player 2 is a bot.  

Explain some of the value in this picture.   
-120[D2]Min mean  
-120 is the min value of these below nodes (0, -22, -81, -120, 67, 99, 9, 67, -51)   
	
62[F3] Max mean  
62 is the max value of the below nodes (62, -90, -21, -25, -3)  

0[C1] Leaf mean
This is a leaf node, the evaluation function was executed on this node  
the value 0.
	
	

### Testing  
![Testing](https://github.com/KDevZilla/ImageUpload/blob/main/KReversi/List_of_Integrate_Test.PNG)  

You can just run all of the unit and integration tests.  
Please make sure it passes all of them, but you can skip BotFightTest because it took about 20 seconds to finish. 

### What can we do to improve 
If we need to improve the program, these are some things we can do to strengthen AI.  
1. Change the data structure of the board from a 2D array to a bitboard.  
2. Include Openbook features; we calculate the evaluation in advance to save it in a file,  
then we simply get the value from it without having to calculate it again.  	
3. XOr hash should be used instead of this hash function.

### Point of interest  
There are many features that I did not intend to include in the first place,   
such as creating a board or a bot, but after developing them,   
I realized that they greatly aided me in finding bugs and fixing programs.

### Reference  
https://www.codeproject.com/Articles/24461/Reversi  
https://www.codeproject.com/Articles/4672/Reversi-in-C  
https://github.com/clarkkev/othello-ai/blob/master/src/othellosaurus/Board.java  
https://github.com/BSmith156/Othello-AI  
https://en.wikipedia.org/wiki/Minimax  
https://en.wikipedia.org/wiki/Alpha%E2%80%93beta_pruning  
https://stackoverflow.com/questions/6468024/how-to-change-combobox-background-color-not-just-the-drop-down-list-part  
https://stackoverflow.com/questions/57124243/winforms-dark-title-bar-on-windows-10  
https://stackoverflow.com/questions/21325661/convert-an-image-selected-by-path-to-base64-string  




	
