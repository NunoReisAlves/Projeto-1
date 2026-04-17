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

`SuperMario\Camera.cs`

A classe Camera trata de tudo o que tem a ver com a camera do jogo, defenindo o zoom, os limites e a sua movimentaçao de acordo com o movimento do personagem

______________
`SuperMario\Constants.cs`

Esta classe foi criada para guardar variaveis constantes

__________________
`SuperMario\FrameSelector.cs`

Classe que controla a animaçao dos personagens, correndo a sua sprite sheet

_____________
`SuperMario\SuperMario.cs`

Classe principal que inicializa e corre o jogo, assim como carregar e desenhar tudo o que for necessário.

______________
`SuperMario\Content`

Pasta usada para guardar os mapas do jogo (`maps`) e os sprites para os personagens e objetos (`sprites`).

______________
`SuperMario\Map`

(TO DO)