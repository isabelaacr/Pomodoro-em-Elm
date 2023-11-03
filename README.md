# Pomodoro
Trabalho da disciplina de Paradigmas de Programação em linguagem Elm. 

# Explicação
https://www.loom.com/share/c985fdcfcf4748a79375e08172663f18?sid=6a047d15-0bef-44d4-9774-0365259739e4

## O que é?
Um timer de 25 minutos que possui um intervalo de 5 minutos, com botão de timer e reset.

## Como foi implementado 
https://elm-lang.org/try

## Referências:
https://elm-lang.org/examples/time
https://package.elm-lang.org/packages/elm/time/latest/
https://ellie-app.com/cjLMnNvFVmSa1
https://discourse.elm-lang.org/
https://elm-lang.org/examples/buttons
https://elm.dev.br/aula_1



#### Define o módulo principal e todas as importações que serão necessárias
module Main exposing (..) 

```elm

module Main exposing (..) -- Define o módulo principal e todas as importações que serão necessárias

import Browser
import Html exposing (Html, button, div, h1, text)
import Html.Events exposing (onClick)
import Html exposing (Html, div, h1, text, button)
import Html.Attributes exposing (class, style)
import Task 
import Time exposing (every, toMinute,toSecond)

```

#### Model (Tipos de dados)

```elm

type alias Model =
    { cont : Int --cont sinaliza a contagem regressiva em segundos
    , clockState : ClockState --estado do relógio (executando,pausado, descanso e resetado)
    }

type ClockState
    = Running
    | Paused
    | Reset
    | Resting

init : () -> (Model, Cmd Msg)
init _ =
    ( Model 1500 Running -- 1500 segundos equivalem a 25 minutos => inicializa com 25 minutos
    , Cmd.none
    )

```

#### Update

``` elm
type Msg
    = Tick --Atualização do relógio
    | ToggleClockState --Alterna entre os estados do relógio
    | ResetTime --Reseta o tempo
    | RestingTime --Inicia período de descanso (5 minutos)

update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
    case msg of
        Tick ->
            if model.cont == 0 then --Verifica se o contador chegou a zero
                ( { model | clockState = Resting, cont = 300 } --Inicializa a contagem com 5 minutos
                , Cmd.none
                )
            else --decrementa o contador de tempo restante
                ( { model | cont = model.cont - 1 }
                , Cmd.none
                )

        ToggleClockState ->
            case model.clockState of
                Running -> --Se o relógio estiver em execução muda de estado para pause
                    ( { model | clockState = Paused }
                    , Cmd.none
                    )

                Paused ->  --Se o relógio estiver parado muda de estado para execução
                    ( { model | clockState = Running }
                    , Cmd.none
                    )

                Reset -> --Depois defino que o reset estabelece para iniciar novamente com 25 minutos
                    ( { model | clockState = Running }
                    , Cmd.none
                    )
                    
                Resting -> --Depois defino que o resting estabelece para iniciar novamente com 5 minutos
                    ( { model | clockState = Running }
                    , Cmd.none
                    )

        ResetTime ->
            ( { model | clockState = Running, cont = 1500 } -- Inicia com 25 minutos
            , Cmd.none
            )
        
        RestingTime ->
            ( { model | clockState = Resting, cont = 300 } -- Inicia com 5 minutos
            , Cmd.none
            )

```

#### Subscriptions

``` elm

subscriptions : Model -> Sub Msg
subscriptions model =
    case model.clockState of
        Running ->
            Time.every 1000 (\_ -> Tick) --1000 milisegundos equivalem a 1 minuto, então faz atualização tick quando completado 1 minuto

        _ ->
            Sub.none

```

#### View

``` elm

view : Model -> Html Msg
view model =
    let
        minutes =
            String.fromInt (model.cont // 60) -- Converte os segundos em minutos

        seconds =
            String.fromInt (Basics.modBy 60 model.cont) -- Calcula os segundos restantes

        pauseButtonText =
            case model.clockState of
                Running ->
                    "Pause"

                Paused ->
                    "Resume"

                Reset ->
                    "Start"
                    
                Resting ->
                     "Resume"
       in
    div [ class "pomodoro-timer", style "text-align" "center" ]  -- Centralize o conteúdo
        [ h1 [ style "text-align" "center" ] [ text "Pomodoro" ]  -- Título "Pomodoro" no centro da página
        , h1 [ style "color" "purple", style "font-size" "36px", style "margin" "0 auto" ] [ text (minutes ++ ":" ++ seconds) ]  -- Estilo para o relógio e centralização horizontal
        , button [ style "background-color" "lightblue", style "color" "black", style "font-size" "18px", style "margin-top" "10px", onClick ToggleClockState ] [ text pauseButtonText ]  -- Estilo para o botão de pausa com margem superior
        , button [ style "background-color" "lightgray", style "color" "black", style "font-size" "18px", style "margin-top" "10px", onClick RestingTime ] [ text "Start Resting" ]
        , button [ style "background-color" "red", style "color" "white", style "font-size" "18px", style "margin-top" "10px", onClick ResetTime ] [ text "RESET" ]  -- Estilo para o botão de reset com margem superior
        ]
        ]
```
