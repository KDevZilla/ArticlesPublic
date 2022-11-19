# Introduction

![Game Animation](https://raw.githubusercontent.com/KDevZilla/Resource/main/SharpWord01.gif)

This is a clone of Josh Wardle, a Welsh software engineer, and his well-known Wordle game.  
I created this game to evaluate my ability to perform simple animations using Windows Form.  

# File Structure

## Game
The game business rule belongs here.
## UI
Everything relates to creating and rendering UI.
## Utility
Relate to the path and Serialize the object.
	
![SharpWord_FileStructure](https://user-images.githubusercontent.com/108615376/202791048-403105e5-b00e-42ac-a41b-a05871fa938e.png)

# The concept


All of the Game logic is in the Game folder

### Alphabet.cs  
This class used to store the character and the result of the Word object.\
The Alphabet object itself does not have the ability to verify the result.\
the possibility  of the result is\
CorrectSpot,
WrongSpot,
NoninThWord

	

### Word.cs  
This class holds a list of alphabets, and its most significant method is **IsCorrect ()**.  
If all of the alphabets in the list are accurate, this method returns true; otherwise, it returns false.  
The InCorrect() method assigns the result to the Alphabet object\
in the lstAlphabet in addition to determining whether the result is accurate.\
We must keep the result in Alphabet so that the UI object may use it to render the tile.

```C#
 public Boolean IsCorrect(Word AnswerWord)
        {
            int i = 0;

            Boolean IsCorrect = true;
            if(lstAlphabet.Count != AnswerWord.GetWordAsString().Length )
            {
                IsCorrect = false;
            }
	    /* Check all of the Alphabet in lstAlphabet
	     If there is at least one Alphabet
	     That the result is not AlphaResult.CorrectSpot
	     This function return false, otherwise return true.
	     */
            for (i = 0; i < lstAlphabet.Count ; i++)
            {
                Alphabet AlphaAnswer = AnswerWord.lstAlphabet[i];
                lstAlphabet[i].Result = AlphaResult.NotinTheWord;

                if(lstAlphabet [i].Character == AlphaAnswer.Character  )
                {
                    lstAlphabet[i].Result = AlphaResult.CorrectSpot;
                } else
                {
                    if(AnswerWord.IsItContainChar (lstAlphabet [i].Character ))
                    {
                        lstAlphabet[i].Result = AlphaResult.WrongSpot;
                    }
                }
                if(lstAlphabet [i].Result == AlphaResult.NotinTheWord ||
                    lstAlphabet [i].Result == AlphaResult.WrongSpot )
                {
                    IsCorrect = false;
                }
            }
            return IsCorrect;

        }
```

### SharpWordGame.cs  
This class contains a list of Words and game information and ISharpWordUI.

These are important methods of this class\
**EnterChar(String KeyData)**  this method accepts the value from the keyboard, then checks if\
the UI was not blocked, block the UI input and then call Operation() after the Operation() method\
was executed, Unblock the UI input.
	
**Operation(String KeyData)**  this method accepts KeyData and then handles the game logic.
 1. If it is the Back key, remove Char then returns.
 2. If it is a RETURN key, SubmitAnswer() then returns;
 3. If it reaches this point, it means it is neither Back nor Return,  
then program Add Character to the word.\
**LoadListWord()** this method will load the list of the word from the file path.\
**InitialGame()** load the list of WOrd from the file then set the answer,\
initial _lstGuessWord then calls the UI object to render the game.\
**CheckAnswer()** to check if the answer is correct.
	
These are the important properties.\
**CurrentGuessWord** returns a Word object. \
Behind the scence, it returns \
the currentwordindex from the _lstGuessWord object.\
**PreviousGuessWord** as its name suggests, returns the previous Guess word.\
**MaxWordGuessAllow** number of the word allow the player to Guess before the game is over.\
**MaxCharInWord** number of characters in the word.\
**CurrentWordIndex** we have a _lstGuessWord to contain all of the words object, \
this property contains the current index of the word.


This is the game state.
```C#
	public enum GameStateEnum
        {
            Playing,
            Finished
        }
```

### Statistics.cs  
This class is responsible for keeping the information of \
the number of times the Player Wins/Loss then calculating the percentage.


![SharpWord_SEQ_Diagram](https://user-images.githubusercontent.com/108615376/202255768-826f7977-9955-4958-8380-cd916ecdbf7e.png)

This image shows a sequence diagram  between UI and the Game object.

### The UI
The code in the UI parts took more than 90% of the time I developed this project because the Windows Form 
is not a CSS, so you cannot just flip an image by writing 5 lines of code.

### ISharpWordUI.cs
This is an interface that provides the methods that the actual UI object needs to implement.
These are the methods.


```C#
        void SetGame(SharpWordGame pGame);
        void CreateTiles();
        void CreateKeyBoard();
        void CreateBoard();
        void RenderWin();
        void RenderLost();
        void RenderIncorrectRow(int pRowIndexIncorrect);
        void RenderCurrentWord(String str);
        void RenderKeyBoard(Dictionary<Char, AlphaResult> pDicTriecChar);
        void RenderAttemptWord(); // Render in case Incorrect answer
        void RemoveChar(int Row, int Col);
        void SetTheme(Theme pTheme);
        Boolean IsFinishProcessing();
        void BlockInput();  // When UI is rendering we don't accept input
        void UnBlockInput(); // After UI has finished rendering, we accept input.
        void ClearUI();
        void ShowStatistics(Statistics statis);
        Boolean IsInputBlocked();
```


### WinFormUI.cs  
This class is the UI class that implements the ISharpWordUI interface.  
These are 3 important objects in this class  
**pnlMain** is a Panel object that contains plnKeys and pnlTitles.  
**plnKeys** is used for rendering the virtual keyboard.  
**pnlTitles** is used for rendering the tiles of the game.  

![UI Control](https://raw.githubusercontent.com/KDevZilla/Resource/main/SharpWorld_Screen04.png)

**CreateTitles()** is a method this class uses for rendering the tiles, 
we simply create arrays of the Labels  
then let pnlTitles add them as controls.  
**CreateKeyBoard()** This method also uses an array of labels as a keyboard key.

**RenderTheme()** this program supports dark and light modes, 
and we use this method to render the game's appearance.  
**GetTheme()** returns the Theme object according to the parameter IsDarkTheme.

```C#
        //We have Dictionary to map the character and the RoundLabel
        Dictionary<String, RoundLabel> DicKeyBoard = new Dictionary<String, RoundLabel>();
        String[] arrKey = { "QWERTYUIOP", "ASDFGHJKL", ">ZXCVBNM<" };
        public void CreateKeyBoard()
        {
            pnlKeys = new Panel();
            DicKeyBoard = new Dictionary<String, RoundLabel>();
           
            int i;
            int j;
            int SpaceBetweenX = 5;
            int SpaceBetweenY = 5;
            Label PreviousKey = null;
            int MaxWidth = 0;
	    //Loop thought row 0 to 2
            for (i = 0; i < arrKey.Length; i++)
            {
                PreviousKey = null;
                //Loop thought all character in each row
                for (j = 0; j < arrKey[i].Length; j++)
                {
                    String cKey = arrKey[i][j].ToString();

                    RoundLabel LblKey = new RoundLabel();
                    LblKey.Font = lblTemplateKey.Font;
                    LblKey.TextAlign = lblTemplateKey.TextAlign;
                    LblKey.AutoSize = lblTemplateKey.AutoSize;

                    int KeyWidth = lblTemplateTile.Width;
                    int KeyLeft = 0;

                    if (i == 1 && j == 0)
                    {
                        KeyLeft = lblTemplateTile.Width / 2;
                    }
                    else
                    {
                        if (PreviousKey == null)
                        {
                            KeyLeft = (j * (LblKey.Width + SpaceBetweenX) + SpaceBetweenX);
                        }
                        else
                        {
                            KeyLeft = PreviousKey.Left + PreviousKey.Width + SpaceBetweenX;
                        }
                    }
		    // > and Enter act the same
                    if (cKey == ">")
                    {
                        cKey = "Enter";
                        KeyWidth = 100;
                    }
		    // < and ? act the same
                    if (cKey == "<")
                    {
                        cKey = "?";
                        KeyWidth = MaxWidth - KeyLeft;
                    }
                
                    LblKey.Text = cKey.ToString();
                    LblKey.Width = KeyWidth;
                    LblKey.Height = lblTemplateKey.Height;
                    LblKey.Top = (i * (LblKey.Height + SpaceBetweenY) + SpaceBetweenY);
                    LblKey.Left = KeyLeft;
                    LblKey.Visible = true;
                    LblKey.Click += LblKey_Click;
                    PreviousKey = LblKey;
                    DicKeyBoard.Add(LblKey.Text, LblKey);
                    pnlKeys.Controls.Add(LblKey);
                    if(j== arrKey [i].Length -1)
                    {
		        //The latest character in a row
			
                        if(LblKey.Left + LblKey.Width > MaxWidth)
                        {
                            MaxWidth = LblKey.Left + LblKey.Width;
			    // Adjust the width
                        }
                    }
                }
            
            }
            pnlKeys.Height = PreviousKey.Top + PreviousKey.Height + SpaceBetweenY;
            pnlKeys.Width = MaxWidth;
        }
```
```C#
        public void CreateTiles()
        {
            int i;
            int j;
            // Loop thought all of the Row
            for (i = 0; i < _Game.MaxWordGuessAllow; i++)
            {
	        // Loop thoguht all of the character in the row;
                for (j = 0; j < this.MaxWordLength; j++)
                {

                    Label labelTile = new Label();
                    labelTile.Height = lblTemplateTile.Height;
                    labelTile.Width = lblTemplateTile.Width;

                    labelTile.Font = lblTemplateTile.Font;
                    labelTile.FlatStyle = FlatStyle.Flat;
                    labelTile.BorderStyle = BorderStyle.FixedSingle;
                    labelTile.TextAlign = ContentAlignment.MiddleCenter;
                    labelTile.Visible = true;
                    labelTile.Text = "";
                    labelTile.Name = GetLableID(i,j);
                    // Add to DicButton
                    DicButton.Add(labelTile.Name, labelTile);
                }
            }

            this.pnlTiles = new DoubleBufferedPanel();

            SetDoubleBuffered(this.pnlTiles);

            this.pnlTiles.Controls.Clear();
            int HeightOffset = 8;
            int WidthOffset = 8;

            List<Label> lstLbl = DicButton.Values.ToList();
            //Loop thought all of the label in DicButton 
	    //To set the position
            for (i = 0; i < lstLbl.Count; i++)
            {
                String Name = lstLbl[i].Name;
                int iTop = int.Parse(Name.Substring(0, 2));
                int iLeft = int.Parse(Name.Substring(2, 2));

                lstLbl[i].Top = iTop * (lstLbl[i].Height + HeightOffset) + HeightOffset * 2;
                lstLbl[i].Left = iLeft * (lstLbl[i].Width + WidthOffset) + WidthOffset;

                this.pnlTiles.Controls.Add(lstLbl[i]);
            }

            Label lastLbl = lstLbl[lstLbl.Count - 1];
            this.pnlTiles.Width = lastLbl.Left + lastLbl.Width + WidthOffset;
            this.pnlTiles.Height = lastLbl.Top + lastLbl.Height + HeightOffset;
            lblAnswer = new RoundLabel();

            lblAnswer.AutoSize = false;
            lblAnswer.Text = _Game.WWordAnswer.GetWordAsString();
            lblAnswer.Top = 100;
            lblAnswer.Width = 160;
            lblAnswer.AutoSize = false;
            lblAnswer.Height = 80;
            lblAnswer.TextAlign = ContentAlignment.MiddleCenter;
            lblAnswer.Left = (this.pnlTiles.Width - lblAnswer.Width) / 2;
            lblAnswer.Visible = false;
            lblAnswer.BringToFront();
            this.pnlTiles.Controls.Add(lblAnswer);
        }
```
```C#
        private  Theme GetTheme(Boolean IsDarkTheme)
        {
            Theme theme = new Theme();
            
            if (IsDarkTheme)
            {
	       //Just set the value in case of DarkMode
                theme.TileNormalBackColor = Color.White;
                theme.TileNormalForeColor = Color.FromArgb(18, 18, 18);
                theme.LabelAnswerBackColor = Color.FromArgb(18, 18, 18);
                theme.LabelAnswerForeColor = Color.White;
                theme.TileCorrectBackColor = Color.FromArgb(106, 170, 100);
                theme.TileCorrectForeColor = Color.White;
                theme.TileNotExistBackColor = Color.FromArgb(58, 58, 60);
                theme.TileNotExistForeColor = Color.White;
                theme.TileNotCorrectPositionBackColor = Color.FromArgb(201, 180, 88);
                theme.TileNotCorrectPositionForeColor = Color.White;

                theme.KeyForeColor = Color.Black;
                theme.KeyBackColor = Color.FromArgb(211, 214, 218);
                theme.BoardBackColor = Color.FromArgb(18, 18, 19);
                theme.BoardForeColor = Color.White;
                theme.ButtonBackColor = Color.White;
                theme.ButtonForeColor = Color.Black;


                theme.PopupFormBackColor = Color.FromArgb(20,18,20);
                theme.IsFormCaptionDarkMode = true;
            }
            else
            {
                // Light mode.
                theme.TileNormalBackColor = Color.White;
                theme.TileNormalForeColor = Color.Black;
                theme.LabelAnswerBackColor = Color.FromArgb(18, 18, 18);
                theme.LabelAnswerForeColor = Color.White;

                theme.TileCorrectBackColor = Color.FromArgb(83, 141, 78);
                theme.TileCorrectForeColor = Color.White;
                theme.TileNotExistBackColor = Color.FromArgb(120, 124, 126);
                theme.TileNotExistForeColor = Color.White;
                theme.TileNotCorrectPositionBackColor = Color.FromArgb(201, 180, 88);
                theme.TileNotCorrectPositionForeColor = Color.White;
                theme.KeyForeColor = Color.Black;
                theme.KeyBackColor = Color.FromArgb(211, 214, 218);
                theme.BoardBackColor = Color.White;
                theme.BoardForeColor = Color.Black;

                theme.ButtonBackColor = Color.Black;
                theme.ButtonForeColor = Color.White;
                
                theme.PopupFormBackColor = Color.FromArgb (240,240,240);
                theme.IsFormCaptionDarkMode = false;

            }
            return theme;
        }
```
```C#
        public void RenderTheme()
        {
            if(pnlMain ==null)
            {
                return;
            }
            // Set BackColor to each panel
            this.pnlMain.BackColor = _CurrentTheme.BoardBackColor;
            this.pnlTiles.BackColor = _CurrentTheme.BoardBackColor;
            this.pnlKeys.BackColor = _CurrentTheme.BoardBackColor;
            Form.BackColor = pnlMain.BackColor;

            lblAnswer._BackColor = _CurrentTheme.LabelAnswerBackColor;
            lblAnswer.Font = lblTemplateTile.Font;
            lblAnswer.ForeColor = _CurrentTheme.LabelAnswerForeColor;

            Color BackColorButton = Color.White;
            Color ForeColor = Color.Black;
            Color BorderColor = Color.Black;
            int i;
            int j;
            //Loop thought all of the label to set ForColor and BackColor
            for (i = 0; i < _Game.MaxWordGuessAllow; i++)
            {
                for (j = 0; j < this.MaxWordLength; j++)
                {

                    Label labelTile = DicButton[GetLableID(i,j)];
                    labelTile.ForeColor = _CurrentTheme.TileNormalForeColor;
                    labelTile.BackColor = _CurrentTheme.TileNormalBackColor;
                }
            }

            for (i = 0; i < this._Game.lstGuessWord.Count; i++)
            {
                BorderStyle borderStyle = BorderStyle.FixedSingle;
                for (j = 0; j < this._Game.lstGuessWord[i].lstAlphabet.Count; j++)
                {
                    if (this._Game.lstGuessWord[i].GetWordAsString().Length < this.MaxWordLength)
                    {
                        BackColorButton = _CurrentTheme.TileNormalBackColor;
                        ForeColor = _CurrentTheme.TileNormalForeColor;
                    }
                    else
                    {
                        BackColorButton = Color.White;
                        ForeColor = Color.White;
                        borderStyle = BorderStyle.None;
			// Set BackColorButton, ForeColor according to the result;
                        switch (this._Game.lstGuessWord[i].lstAlphabet[j].Result)
                        {
                            case AlphaResult.CorrectSpot:
                                BackColorButton = _CurrentTheme.TileCorrectBackColor;
                                ForeColor = _CurrentTheme.TileCorrectForeColor;
                                break;
                            case AlphaResult.WrongSpot:
                                BackColorButton = _CurrentTheme.TileNotCorrectPositionBackColor;
                                ForeColor = _CurrentTheme.TileNotCorrectPositionForeColor;
                                break;
                            case AlphaResult.NotinTheWord:
                                BackColorButton = _CurrentTheme.TileNotExistBackColor;
                                ForeColor = _CurrentTheme.TileNotExistForeColor;
                                break;
                            default:
                                throw new Exception("Wrong value");
                        }
                    }

                    Label labelTile = DicButton[GetLableID(i,j)];
                        labelTile.BackColor = BackColorButton;
                        labelTile.BorderStyle = borderStyle;
                        labelTile.ForeColor = ForeColor;
                }
                
            }
            for (i = 0; i < arrKey.Length; i++)
            {
                for (j = 0; j < arrKey[i].Length; j++)
                {
                    String cKey = arrKey[i][j].ToString();
                    if (cKey == ">")
                    {
                        cKey = "Enter";  
                    }
                    if (cKey == "<")
                    {
                        cKey = "?";
                    }

                    RoundLabel labelKey = new RoundLabel();
                    labelKey = DicKeyBoard[cKey];
                    labelKey.ForeColor = _CurrentTheme.KeyForeColor;
                    labelKey._BackColor = _CurrentTheme.KeyBackColor;
                }

            }
            Utility.Utility.MakeFormCaptionToBeDarkMode(this.Form, _CurrentTheme.IsFormCaptionDarkMode);
        }
```

### MainUI.cs  
This class contains a UI object and a game object, 
it acts like glue between those 2 objects.
	
## Render Animation
This project uses Transitions.dll to help with the rendering part. 
The Transitions.dll has a Transition class which can help us make a smooth animation. 

We use Transitions because we need to find the rate of change of a parameter over time.

Supposing we would like to move a label that its left property is 10  
then we need to move its left position to 40 within 5 seconds.

In each second the left property will be increased by 6,   
after 5 seconds have passed the left property would be 40 as we expected.

The problem is it will not look so smooth  
because the rate of change is the same in each step.  
In nature when the object moves, it does not move at the same rate.  
If our program moves the object at the same rate, it will look strange. 
	

This site has information on how Transition work https://easings.net/  
It uses CSS as an example but it also provides us with a Math function.

### TransitionExtend.cs 
This is a class that inherits from Transitions.Transition class  
We use this class to set the property of the object,  
the property of the object will keep being updated until it reaches the goal automatically.  

The dll we used is Transistion.dll  
You can check for more information here.  
https://github.com/UweKeim/dot-net-transitions



![SharpWordTransnstions](https://user-images.githubusercontent.com/108615376/202790222-c2c59888-23ed-450c-b251-b9e3e5f37af6.gif)

This is an example of the code that uses Transitions to move the label  
vs the non-transitions moving.  
You can look into the code in frmSampleTransitions.cs 


```C#
	Timer timerMove = new Timer();
	private int DestinationLeft = 0;
	private int StepSize = 10;
	private int NumberofStep = 40;
	int iCount = 0;
	Utility.TimeMeasure timeMeasure = null;
	private void btnRun_Click(object sender, EventArgs e)
	{
		timeMeasure = new Utility.TimeMeasure();
		timeMeasure.Start();
		lblTran.Left = 30;
		lblNonTran.Left = 30;
		DestinationLeft = lblTran.Left + (StepSize  * NumberofStep);
		StartTranMoving();
		StartNonTranMoving();
	}
	
	private void StartTranMoving()
	{
		/*
		 If timer.Interval is precious this value supposed to be
		 1000.
		 We use 1285 instead because it tooks about 1.285 seconds 
		 for a timer to tick 40 times using interval 25.         
		*/
		int Millisecond = 1285;
		Transitions.Transition tran = new Transitions.Transition(new TransitionType_EaseInEaseOut(Millisecond));
		//Just use method add
		//The parameter are object, propertyname, destination of the property value.
		tran.add(lblTran, "Left", DestinationLeft);
		tran.run();
	}
	
	private void StartNonTranMoving()
	{
		if (timerMove != null)
		{
			timerMove.Enabled = false;
		}
		timerMove = new Timer();
		timerMove.Enabled = true;
		timerMove.Interval = 25;
		timerMove.Tick += TimerMove_Tick;

	}
	private void TimerMove_Tick(object sender, EventArgs e)
	{
		lblNonTran.Left += StepSize; //Walking
		if (lblNonTran.Left >= DestinationLeft) //Reach destination
		{
			timerMove.Enabled = false;
			timeMeasure.Finish();
			this.Text = "Time takes (seconds) " + timeMeasure.TimeTakes.TotalSeconds;
		}
	}
```

## Render tiles when the character is entered. 
We use **TransitionHelper.PopLabel()** method. 
The logic in this method() is,  
 1. Keep the Original position of the label to the variables (the name is OriLeft, OriTop, OriHeight, OriWidth) 
 2. Decrease the size of the label by moving it to the bottom right position a little bit and also decrease its size 
 3. Use the TransitionExtend object to update the Left, Top, Width, and Height properties to their original values.

```C#
        public static  void PopLabel(Label plabel, String ptext,Color pforecolor, int iTransactionTime)
        {
            plabel.ForeColor = pforecolor;
            TransitionExtend t = new TransitionExtend(new TransitionType_EaseInEaseOut(iTransactionTime));
            //Keep Original position
            int OriLeft = plabel.Left;
            int OriTop = plabel.Top;
            int OriHeight = plabel.Height;
            int OriWidth = plabel.Width;
            
            //Change size and position a little bit
            plabel.Left += 5;
            plabel.Top += 5;
            plabel.Height -= 10;
            plabel.Width -= 10;
            
            plabel.Text = ptext;
            plabel.ForeColor = Color.Black;
            plabel.Tag = OriLeft;
            
            // use TransitionExtend object to change the size and position back within a specific time.
            t.add(plabel, "Left", OriLeft);
            t.add(plabel, "Top", OriTop);
            t.add(plabel, "Width", OriWidth);
            t.add(plabel, "Height", OriHeight);
            t.Tag = plabel;

            t.run();
            t.TransitionCompletedEvent += T_TransitionCompletedEvent;

        }

        private static  void T_TransitionCompletedEvent(object sender, Transition.Args e)
        {
            // After Transition has completed try to Adjust position.
            Label lbl = (Label)((TransitionExtend)sender).Tag;
            try
            {
                AdjustLeftProperty(lbl, (int)lbl.Tag);
            }catch (Exception ex)
            {
                //Do nothing
            }
        }
        private static  void AdjustLeftProperty(Label plabel, int Left)
        {
            if (plabel.InvokeRequired)
            {
                //Handle in case the non UI Thrad need to set the property of the control.
                plabel.Invoke(new Action<Label, int>(AdjustLeftProperty), plabel,Left);
            }
            else
            {
                plabel.Left = Left;
            }
        }
```
	
## Render tiles when the answer is incorrect.   
We use the **RenderShake()** method. 
Just loop through all of the labels that we use for tiles then update the Left value and also use  
Sleep the thread for 2 milliseconds every time the labels are moving. 

```C#
        public void RenderShake(int pRowIndexIncorrect)
        {
            List<Label> lstB = new List<Label>();

            int i;
            int j;
            int iValueChange = 1;
            int iLoop = 0;
            int[] arrLeft = new int[5];
            for (i = 0; i <= 4; i++)
            {
                lstB.Add(DicButton[GetLableID(pRowIndexIncorrect,i)]);
                arrLeft[i] = DicButton[GetLableID(pRowIndexIncorrect, i)].Left;
            }

            for (iLoop = 0; iLoop < 4; iLoop++)
            {
                for (i = 1; i <= 10; i++)
                {
                    for (j = 0; j < lstB.Count; j++)
                    {
                        lstB[j].Left += iValueChange;

                    }
                    if (i % 2 == 0)
                    {
                        System.Threading.Thread.Sleep(2);
                        Application.DoEvents();
                    }
                }
                iValueChange *= -1;
            }
            for (i = 0; i <= 4; i++)
            {
                
                lstB[i].Left = arrLeft[i];
            }

        }
```


## Render Tiles when the player lost. 

![SharpWorld_Lost](https://user-images.githubusercontent.com/108615376/202787704-72a161e3-bbb3-480a-98c9-d60113ca190a.png)

This is a thing we would like to show  
But we cannot use a RoundLabel to show due to its limit  
to display the corner color correctly in case the RoundLabel is on top  
of the other control.


![SharpWorld_RoundLabelProblem](https://user-images.githubusercontent.com/108615376/202787940-8acdee1b-767d-4176-a12d-4ef827cdb003.png)

This picture shows a problem with RoundLabel control.

What we need to do is still need to have a RoundLabel control,  
We named it lblAnswer, but we don't have the intention to show it. 

We hide the first 3 rows of label titles and the lblAnswer.  
Then draw the images of those 15 label titles then draw the rectangle object  
using the information from lblAnswer. 

We do it in **Panel1_Paint** event. 

```C#
	private void Panel1_Paint(object sender, PaintEventArgs e)
	{
		if (IsLost)
		{

			int i;
			int j;
			for (i = 0; i <= 2; i++) // 3 Rows
			{
				for (j = 0; j <= 4; j++) // All of Label in each Row
				{
					Label LTemp = DicButton[GetLableID(i, j)];
					LTemp.Visible = false; // Hide
					DrawLabel(e.Graphics, LTemp); // Draw it on panel
				}
			}
			
			Rectangle rPosition = lblAnswer.ClientRectangle;
			rPosition.X += lblAnswer.Left;
			rPosition.Y += lblAnswer.Top;
                        // Draw lblAnswer on Panel
			using (var graphicsPath = lblAnswer._getRoundRectangle(rPosition))
			{

				e.Graphics.SmoothingMode = SmoothingMode.AntiAlias;
				using (var brush = new SolidBrush(lblAnswer._BackColor))
					e.Graphics.FillPath(brush, graphicsPath);
				using (var pen = new Pen(lblAnswer._BackColor, 1.0f))
					e.Graphics.DrawPath(pen, graphicsPath);
				TextRenderer.DrawText(e.Graphics, lblAnswer.Text, lblAnswer.Font, rPosition, lblAnswer.ForeColor);
			}
			return;
		}

	}
```

## Render Tiles when the player won. 



We use **SwapLabel.DanceLabel()** methods.  
This method uses a timer to trigger the **Ti_TickV2()** method.  
**Ti_TickV2()** select the current tile from indexBtnMove field  
then uses TransitionExtend to change the Top and BackColor properties of the label.  
then indexBtnMove++ so that next time this method will move the next tile  
	
```C#
        public void DanceLabel(List<Label> pLabelList, int pTranstitionTime, int pNumberofLoop)
        {
            labelList = pLabelList;

            NumberofLoop = pNumberofLoop;
            TranstitionTime = pTranstitionTime;
            Timer Ti = new Timer();
            Ti.Interval = 200;
            Ti.Tick += Ti_TickV2;
            Ti.Enabled = true;

        }
        int iCountLoop = 0;
        int indexBtnMove = 0;
        int NumberofLoop = 1;
        int TranstitionTime = 200;
        private void Ti_TickV2(object sender, EventArgs e)
        {
            //Get label at Current index.
            Label label = this.labelList[indexBtnMove];
            TransitionExtend transition = new TransitionExtend(new TransitionType_EaseInEaseOut(TranstitionTime));
            //Set properties Top, BackColor on transition object
	    //For Top property we use the current position - 20 to make to go upper.
            transition.add(label, "Top", label.Top - 20);
            transition.add(label, "BackColor", Color.Teal);

            //To make label goes down we set label.Top + 5
            TransitionExtend tBack = new TransitionExtend(new TransitionType_EaseInEaseOut(TranstitionTime));
            tBack.add(label, "Top", label.Top + 5);

            //To make label goes back up to the original position.
            TransitionExtend tBack2 = new TransitionExtend(new TransitionType_EaseInEaseOut(450));
            tBack2.add(label, "Top", label.Top);

            //transition->tback->tback2
	    //Go up -> Go Down ->Go to the orginal position.
            transition.Childs = new List<TransitionExtend>();
            transition.Childs.Add(tBack);
            tBack.Childs = new List<TransitionExtend>();
            tBack.Childs.Add(tBack2);

            transition.run();
            indexBtnMove++;
            //If we finished the latest Label 
            if (indexBtnMove >= this.labelList.Count)
            {
                iCountLoop++;
                indexBtnMove = 0;
		//If we reached the last loop then
                if (iCountLoop > NumberofLoop)
                {
                    Timer thisTimer = (Timer)sender;
                    thisTimer.Enabled = false;
                    Complete?.Invoke(this, new EventArgs());
		    //Complete event.
                }
                return;
            }
        }
```

## Render Tiles Flipping.
We use **SwapLabel.SwapNotUsingTimer()**  
The concept of this method is just draw the label  
using DrawImage(), the second argurment of this method look like this    

Point[] destinationPoints = {  
      new Point(0, 0),    // destination for upper-left point of original  
      new Point(100, 0),  // destination for upper-right point of original  
      new Point(0, 100)};// destination for lower-left point of original  

With each step that we draw, we will calculate the position of the y-axis.  
so that we can decrease the size of the image until its height < 0  

then we will change its back color  
and increase the size of the image until it reaches its original size.  


Step 1-6 is to calculate the position of the label  
We need to draw. 
Step7 is the actual step that draw  
the label image.  

 		 
1. Store FirstY= Label.Top for the fist loop
2. Hide the label.
3. Use label.DrawToBitmap() to create a new Bitmap.
4. Calculate the NewDes point.
5. Set pp.DrawImage property, NewDes (pp is DoubleBufferedPanel that we use)
6. call pp.Invalidate() so that the Paint_SwapLabel() method will be called.
7. In Paint_SwapLabel() method, it will read the information from the panel 
then draw the image.

![Label Flip](https://github.com/KDevZilla/Resource/blob/main/SharpWorld_SwapLabel.png)

  
```C#
	public void SwapNotUsingTimer(SharpWord.UI.DoubleBufferedPanel  pp, List<Label> pLabelList, int pMilisecondThreadSleep)
	{
		pp.Paint += Paint_SwapLabel;
		try
		{
			labelList = pLabelList;

			if (pMilisecondThreadSleep == -1)
			{
				pMilisecondThreadSleep = 15;
			}
			int MilisecondSleepBetwenPile = pMilisecondThreadSleep * 15;
			int MilisecondThreadSleepBackCard = Convert.ToInt32(pMilisecondThreadSleep * 1.5);

			CurrentarrLabelSwapIndex = 0;
			Boolean IsProcessing = true;
			iYChange = 2;
			int iTemp = 0;
			while (IsProcessing)
			{
				if (IsShowBackCard)
				{
					System.Threading.Thread.Sleep(MilisecondThreadSleepBackCard );
				}
				else
				{
					System.Threading.Thread.Sleep(pMilisecondThreadSleep);
				}

				 Application.DoEvents();

				Label L = labelList[CurrentarrLabelSwapIndex];
				LabelAttribute NewAttribute = (LabelAttribute)L.Tag;
				int inumerator = L.Height / iYChange;

				iTemp++;
				iTemp = 0;
				if (LoopCount == 0)
				{
					L.Visible = false;
				}

				Bitmap b = new Bitmap(L.Width, L.Height);
				L.DrawToBitmap(b, new Rectangle(0, 0, b.Width, b.Height));

				Image image = b;

				LoopCount++;

				Point[] NewDes = {
new Point(L.Left , L.Top ),   // destination for upper-left point of
				  // original
new Point(L.Left + L.Width, L.Top ),  // destination for upper-right point of
				  // original
new Point(L.Left , L.Top + L.Height)};  // destination for lower-left point of
									// original


				if (IsShowBackCard)
				{
					NewDes[0].Y = L.Top + (iYChange * (inumerator - LoopCount));
					L.BackColor = NewAttribute.BackColor;
					L.ForeColor = NewAttribute.ForeColor;
				}
				else
				{
					NewDes[0].Y = L.Top + (iYChange * LoopCount);
				}

				if (LoopCount == 1)
				{
					FirstY = L.Top;// NewDes[0].Y;
				}
				NewDes[1].Y = NewDes[0].Y;
				if (IsShowBackCard)
				{
					NewDes[2].Y = L.Top + L.Height - (iYChange * (inumerator - LoopCount));
				}
				else
				{
					NewDes[2].Y = L.Top + L.Height - (iYChange * LoopCount);
				}
				pp.DrawImage = image;
				pp.NewDes = NewDes;
				pp.Invalidate();
			   
				if (NewDes[0].Y > NewDes[2].Y)
				{
					IsShowBackCard = true; //Its is a time to flip.
				}
				if (NewDes[0].Y <= FirstY)
				{
					if (IsShowBackCard)
					{
						LoopCount = 0;
						L.Visible = true;
						Application.DoEvents();
						// If CurrentLabel is not the last label yet
						if (CurrentarrLabelSwapIndex < labelList.Count - 1)
						{
							System.Threading.Thread.Sleep(MilisecondSleepBetwenPile);
							IsShowBackCard = false;
							CurrentarrLabelSwapIndex++;
						}
						else
						{
							IsProcessing = false; //Finish becasue it is the last label
						}

					}
				}
			}
		} catch (Exception ex)
		{
			throw;
		}
		finally {
			pp.Paint -= Paint_SwapLabel; // Clear the event handler
		}
		Complete?.Invoke(this, new EventArgs()); //Raise event that it already finish.
		return;
	}
	
	private void Paint_SwapLabel(object sender, PaintEventArgs e)
        {
            //The actual method that draw
            try
            {
                SharpWord.UI.DoubleBufferedPanel panel = (SharpWord.UI.DoubleBufferedPanel)sender;
                e.Graphics.Clear(panel.BackColor);
                e.Graphics.DrawImage(panel.DrawImage, panel.NewDes);
            }catch (Exception ex)
            {
                //Do nothing in case of image is swapping too fast, it migth throw an exception
            }
            

        }
```

## Testing

You can run a unit or integration test on a test project.    
There are not many test methods here because most of the code in this project  
relate to the UI and animation which difficult to automate tests.  

![SharpWord_UnitTest](https://user-images.githubusercontent.com/108615376/202790910-21bce08e-8bf4-43cb-ba65-b672473f9d1c.png)


## Points of Interest  
I tried searching but couldn't find any examples of flipping images and other animations,  
so I created my own function.  
I try to use Timer, Thread.Sleep(), Application.DoEvents()   
then tuning then changing the value and seeing the result.  
The thing is, each of these methods has drawbacks, and the code appears complicated.

I will be more than happy if you use this code,  
but I also hope you discover a more effective technique if you need to flip an image in your application.

### Reference code  
ToggleCheckbox original code  
https://stackoverflow.com/questions/38431674/toggle-switch-control-in-windows-forms

Round Control  
https://stackoverflow.com/questions/42627293/label-with-smooth-rounded-corners

Transitions  
https://www.nuget.org/packages/dot-net-transitions/

### Reference  
Wordle  
https://www.nytimes.com/games/wordle/index.html
