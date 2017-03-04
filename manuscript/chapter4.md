# Chapter 3: Elm

## HTTP Package

```bash
cd web/elm
elm-package install elm-lang/http
```

## Elm Front-end

```elm
module Main exposing (..)

import Html exposing (..)
import Html.Attributes exposing (class)
import Html.Events exposing (onClick)
import Http
import Json.Decode as Decode
import Json.Encode as Encode


-- MAIN


main : Program Never Model Msg
main =
    Html.program
        { init = init
        , view = view
        , update = update
        , subscriptions = subscriptions
        }



-- MODEL


type alias Model =
    { players : List Player
    }


type alias Player =
    { username : String
    , score : Int
    }


init : ( Model, Cmd Msg )
init =
    ( initialModel, performPlayerFetch )


initialModel : Model
initialModel =
    { players = [] }



-- UPDATE


type Msg
    = NoOp
    | FetchPlayers (Result Http.Error (List Player))
    | CreatePlayer (Result Http.Error Player)
    | CreatePlayerButton


update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        NoOp ->
            ( model, Cmd.none )

        FetchPlayers (Ok newPlayers) ->
            ( { model | players = newPlayers }, Cmd.none )

        FetchPlayers (Err _) ->
            ( model, Cmd.none )

        CreatePlayer (Ok player) ->
            ( model, performPlayerFetch )

        CreatePlayer (Err _) ->
            ( model, performPlayerFetch )

        CreatePlayerButton ->
            ( model, performPlayerCreation )



-- JSON Decoders


decodePlayerData : Decode.Decoder Player
decodePlayerData =
    Decode.map2 Player
        (Decode.field "username" Decode.string)
        (Decode.field "score" Decode.int)


decodePlayersList : Decode.Decoder (List Player)
decodePlayersList =
    Decode.list decodePlayerData


decodePlayersFetch : Decode.Decoder (List Player)
decodePlayersFetch =
    Decode.at [ "data" ] decodePlayersList



-- JSON Encoding


defaultPlayer : Encode.Value
defaultPlayer =
    Encode.object
        [ ( "player"
          , Encode.object
                [ ( "username", Encode.string "Default" )
                , ( "score", Encode.int 0 )
                ]
          )
        ]



-- HTTP


fetchPlayers : Http.Request (List Player)
fetchPlayers =
    Http.get "/api/players" decodePlayersFetch


performPlayerFetch : Cmd Msg
performPlayerFetch =
    fetchPlayers |> Http.send FetchPlayers


createPlayer : Http.Request Player
createPlayer =
    Http.post "/api/players" (Http.jsonBody defaultPlayer) decodePlayerData


performPlayerCreation : Cmd Msg
performPlayerCreation =
    createPlayer |> Http.send CreatePlayer



-- SUBSCRIPTIONS


subscriptions : Model -> Sub Msg
subscriptions model =
    Sub.none



-- VIEW


view : Model -> Html Msg
view model =
    div []
        [ ul [] (List.map viewPlayer model.players)
        , createPlayerButton
        ]


viewPlayer : Player -> Html Msg
viewPlayer player =
    li []
        [ p [] [ text player.username ]
        , p [] [ text (toString player.score) ]
        ]


createPlayerButton : Html Msg
createPlayerButton =
    button [ class "btn btn-primary", onClick CreatePlayerButton ] [ text "Create Default Player" ]
```
