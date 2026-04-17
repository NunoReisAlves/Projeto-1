Super Mario
============

Autor
-------
https://github.com/haitrr/super-mario-bros-monogame.git

haitrr


Ideia Geral
------------

1. O jogo é baseado em sprites que representam os objetos que são desenhadas no ecrã através do draw
2. O movimento funciona através da alteração da posição com base na velocidade, sendo influenciado pelo input (teclas) e pela física (ex: gravidade)
3. O movimento é feito através de variáveis de velocidade (xVelocity, yVelocity) que vão sendo alteradas ao longo do tempo
4. O input do jogador é tratado com Keyboard.GetState().IsKeyDown, que controla ações como o movimento do Mario
5. As colisões são feitas com base em retângulos (positionRectangle, sourceRectangle), que representam a área dos objetos no jogo
6. O jogo segue o loop Update/Draw do MonoGame:
- Update trata da lógica (movimento, estados, física)
- Draw trata de desenhar os sprites no ecrã
7. Todos estes elementos (sprites, movimento, input e colisões) funcionam em conjunto para definir o comportamento do jogo

Pontos Fortes
--------------

1. Há organização das scripts
2. São usadas classes
3. Loop de jogo Update/Draw segue o padrão em monogame e evita muitos bugs no futuro

Pontos Fracos
---------------

1. O jogo está incompleto e algum código até está documentado por estar incompleto 
Exemplo
```cs
//if (KB_curState.IsKeyDown(Keys.G) && !KB_preState.IsKeyDown(Keys.G))
//{
//    if (level == Level.Fire)
//    {
//        if (Game.FireBalls.Count < 2)
//        {
//            Game.Add_NewFireball();
//            state = State.ThrowFire;
//        }
//    }
//}
```
Por cause disto não dá para usar bolas de fogo.

2. Há também código que não está documentado mas não está a ser usado (Script das Fireballs), fazendo com que se esteja a gastar memória desnecessariamente:
```cs
{
     switch (state)
     {
         case State.Exploding:
             xVelocity = yVelocity = 0;
             if (Explode.curFrameID < 2)
                 sourceRectangle = Explode.GetFrame(ref StateTimer);
             else
                 canRemove = true;
             break;
         case State.Bouncing:
             sourceRectangle = Bounce.GetFrame(ref StateTimer);
             yVelocity += fall_velocity;
             break;
         default:
             sourceRectangle = Bounce.GetFrame(ref StateTimer);
             break;
     }
     positionRectangle.Width = sourceRectangle.Width;
     positionRectangle.Height = sourceRectangle.Height;
 }
```
3. Falta de gamestates. Não há game over, menu, pausa nem mais que 1 nível por isso dá problemas ao jogar. Se morrer e quiser repetir tem-se que abrir e fechar o jogo
4. Mesmo tendo o ponto positivo de haver organização, esta organização não está organizada da melhor maneira. A lógica está concentrada nas mesmas classes (ex: Mario trata de input, física e render), o que torna o código difícil de manter. Idealmente, cada sistema devia ter uma função específica (input, física, colisões). Pode-se alterar de maneira a que cada sistema tem uma função (input, física, colisões). O código fica mais organizado e fácil de manter
5. Controlos pouco responsivos, provavelmente relacionado com os cálculos de acelaração de movimento
6. As colisões estão espalhadas entre as várias classes, isto torna mais difícil de organizar e corrigir código
7. Há alguma repetição de código que podia ser generalizada e reaproveitada em vez de estar sempre a reescrever-se o mesmo

Estrutura
----------

`Camera.cs`

A classe Camera trata de tudo o que tem a ver com a camera do jogo, defenindo o zoom, os limites e a sua movimentaçao de acordo com o movimento do personagem

```cs
public class Camera
    {
        #region Fields

        public float zoom { get; set; } //zoom ratio
        public Matrix transform { get; set; } //transform matrix
        protected Vector2 position;     //camera's position
        public int leftRectrict { get; private set; }   //restrict to go left
        #endregion

        #region Properties

        #endregion

        #region Constructor

        public Camera()
        {
            zoom = 2.0f;        //zoom 2
            position = Vector2.Zero; //begin at begin of map
            leftRectrict = 0;   //restrict left at 0
        }
        public Vector2 Position
        {
            get { return position; }
            set { position = value; }
        }
        #endregion

        #region Methods
        public void updateLookPosition(Rectangle mario)
        {
            position.X = mario.Left- Constants.viewWidth/(2*zoom) ;     //make sure mario in the center
            position.Y = 0; //camera only move horizontal
            if (position.X <= leftRectrict) position.X = leftRectrict;  //prevent camera from go left
            leftRectrict = (int)position.X; //update restrict
        }
        public void Update()
        {
            transform = Matrix.CreateTranslation(-position.X, -position.Y, 0) *     //translation matrix
                            Matrix.CreateScale(new Vector3(zoom, zoom, 1)); //scale matrix
        }

        #endregion
    }
```
______________
`Constants.cs`

Esta classe foi criada para guardar variaveis constantes

```cs
public static class Constants
    {
        public static int viewWidth=512;
        public static int viewHeight=450;
        public static float max_walk_velocity = 2.5f;
        public static float max_run_velocity = 4;
        public static float jump_velocity = -3;
        public static float fall_velocity = 0.15f;
        public static float max_y_velocity = 5;
        public static int timeBetweenToFireBall = 200;
        public static int brickBreakingTime = 500;
        public static float walk_accelerate = 0.075f;
        public static float run_accelerate = 0.1f;
    }
```
__________________
`FrameSelector.cs`

Classe que controla a animaçao dos personagens, correndo a sua sprite sheet

```cs
class FrameSelector
    {
        //animation frame selector
        public List<Rectangle> Frames;
        private float Fps;
        public int curFrameID;
        private bool LoopBack;

        public FrameSelector(float fps, List<Rectangle> frames, bool loop = true)
        {
            this.Fps = fps;
            this.Frames = frames;

            this.curFrameID = 0;
            this.LoopBack = loop;
        }

        public Rectangle GetFrame(ref float dt)
        {
            if (dt > Fps)
            {
                dt = 0;

                //increase to next frame to be ready
                curFrameID++;
                if (curFrameID >= Frames.Count && LoopBack)
                    curFrameID = 0; //reset if exceed animation                
            }

            //create source rectangle to draw
            return Frames[curFrameID];
        }
    }
```
_____________
`SuperMario.cs`

Classe principal que inicializa e corre o jogo, assim como carregar e desenhar tudo o que for necessário.

```cs
public class SuperMario : Game
    {
        GraphicsDeviceManager graphics;
        SpriteBatch spriteBatch;
        Map currentMap;
        Camera camera;
        public SuperMario()
        {
            graphics = new GraphicsDeviceManager(this);
            Content.RootDirectory = "Content";
            graphics.PreferredBackBufferHeight = Constants.viewHeight; //set screen width and height
            graphics.PreferredBackBufferWidth = Constants.viewWidth;
            currentMap = new Map1_1();  //set current map
        }

        /// <summary>
        /// Allows the game to perform any initialization it needs to before starting to run.
        /// This is where it can query for any required services and load any non-graphic
        /// related content.  Calling base.Initialize will enumerate through any components
        /// and initialize them as well.
        /// </summary>
        protected override void Initialize()
        {
            // TODO: Add your initialization logic here
            //camera = new Camera(new Viewport(100,100,Constants.viewWidth,Constants.viewHeight));
            camera = new Camera(); //create camera
            base.Initialize();
        }

        /// <summary>
        /// LoadContent will be called once per game and is the place to load
        /// all of your content.
        /// </summary>
        protected override void LoadContent()
        {
            // Create a new SpriteBatch, which can be used to draw textures.
            spriteBatch = new SpriteBatch(GraphicsDevice);
            currentMap.loadContent(Content);   //load textures of current map
            // TODO: use this.Content to load your game content here
        }

        /// <summary>
        /// UnloadContent will be called once per game and is the place to unload
        /// game-specific content.
        /// </summary>
        protected override void UnloadContent()
        {
            // TODO: Unload any non ContentManager content here
        }

        /// <summary>
        /// Allows the game to run logic such as updating the world,
        /// checking for collisions, gathering input, and playing audio.
        /// </summary>
        /// <param name="gameTime">Provides a snapshot of timing values.</param>
        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed || Keyboard.GetState().IsKeyDown(Keys.Escape))
                Exit();

            // TODO: Add your update logic here
            currentMap.update(camera,gameTime); //update camera
            camera.Update();        //update map
            base.Update(gameTime);
        }

        /// <summary>
        /// This is called when the game should draw itself.
        /// </summary>
        /// <param name="gameTime">Provides a snapshot of timing values.</param>
        protected override void Draw(GameTime gameTime)
        {
            GraphicsDevice.Clear(Color.CornflowerBlue);
            spriteBatch.Begin(SpriteSortMode.BackToFront, BlendState.AlphaBlend, null, null, null, null, camera.transform);
            // TODO: Add your drawing code here
            currentMap.draw(spriteBatch); //draw map
            spriteBatch.End();
            base.Draw(gameTime);
        }
    }
```