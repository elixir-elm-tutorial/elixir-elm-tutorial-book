# Chapter 4: Building the Sign Up Page

Now that we have the ability to create, read, update, and delete players using
our API, let's make a mental shift into how we want users to interact with our
application. The back-end is solid, and we have our initial Elm application,
and we even have our Phoenix HTML pages from when we generated our Player
resources.

This is where things get interesting, because we're going to have to make some
choices about which parts of our stack will be responsible for the things we're
building. But don't worry, because we're going to cover it all in this chapter!

## Preview

![Preview](images/preview.png)

## Source

```elm
module Main exposing (..)

import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (onClick, onInput, onSubmit)
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
    , newUsername : String
    , errors : String
    }


type alias Player =
    { userId : Int
    , username : String
    , score : Int
    }


init : ( Model, Cmd Msg )
init =
    ( initialModel, performPlayerFetch )


initialModel : Model
initialModel =
    { players = []
    , newUsername = ""
    , errors = ""
    }



-- UPDATE


type Msg
    = NoOp
    | Input String
    | Clear
    | PlayerCreate String
    | PlayerCreateHandler (Result Http.Error Player)
    | PlayerFetch
    | PlayerFetchHandler (Result Http.Error (List Player))
    | PlayerUpdate Player
    | PlayerUpdateHandler (Result Http.Error Player)
    | PlayerDelete Player
    | PlayerDeleteHandler (Result Http.Error Player)


update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        NoOp ->
            ( model, Cmd.none )

        Input username ->
            ( { model | newUsername = username }, Cmd.none )

        Clear ->
            ( { model | newUsername = "" }, Cmd.none )

        PlayerCreate username ->
            ( model, performPlayerCreation username )

        PlayerCreateHandler (Ok player) ->
            ( model, performPlayerFetch )

        PlayerCreateHandler (Err httpError) ->
            ( { model | errors = httpError |> toString }, performPlayerFetch )

        PlayerFetch ->
            ( model, performPlayerFetch )

        PlayerFetchHandler (Ok newPlayers) ->
            ( { model | players = newPlayers }, Cmd.none )

        PlayerFetchHandler (Err httpError) ->
            ( { model | errors = httpError |> toString }, Cmd.none )

        PlayerUpdate player ->
            ( model, (performPlayerUpdate player) )

        PlayerUpdateHandler (Ok player) ->
            ( model, performPlayerFetch )

        PlayerUpdateHandler (Err httpError) ->
            ( { model | errors = httpError |> toString }, performPlayerFetch )

        PlayerDelete player ->
            ( model, (performPlayerDelete player) )

        PlayerDeleteHandler (Ok player) ->
            ( model, performPlayerFetch )

        PlayerDeleteHandler (Err httpError) ->
            ( { model | errors = httpError |> toString }, performPlayerFetch )



-- JSON Decoding


decodePlayersFetch : Decode.Decoder (List Player)
decodePlayersFetch =
    Decode.at [ "data" ] decodePlayersList


decodePlayersList : Decode.Decoder (List Player)
decodePlayersList =
    Decode.list decodePlayerData


decodePlayerData : Decode.Decoder Player
decodePlayerData =
    Decode.map3 Player
        (Decode.field "id" Decode.int)
        (Decode.field "username" Decode.string)
        (Decode.field "score" Decode.int)



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


newPlayer : String -> Encode.Value
newPlayer username =
    Encode.object
        [ ( "player"
          , Encode.object
                [ ( "username", Encode.string username )
                , ( "score", Encode.int 0 )
                ]
          )
        ]



-- API


endpoint : String
endpoint =
    "/api/players"


playerCreation : String -> Http.Request Player
playerCreation username =
    Http.post endpoint (Http.jsonBody (newPlayer username)) decodePlayerData


performPlayerCreation : String -> Cmd Msg
performPlayerCreation username =
    username
        |> playerCreation
        |> Http.send PlayerCreateHandler


playerFetch : Http.Request (List Player)
playerFetch =
    Http.get endpoint decodePlayersFetch


performPlayerFetch : Cmd Msg
performPlayerFetch =
    playerFetch |> Http.send PlayerFetchHandler


playerUpdate : Player -> Http.Request Player
playerUpdate player =
    Http.request
        { method = "PUT"
        , headers = []
        , url = "/api/players/" ++ (toString player.userId)
        , body = Http.jsonBody defaultPlayer
        , expect = Http.expectJson decodePlayerData
        , timeout = Nothing
        , withCredentials = False
        }


performPlayerUpdate : Player -> Cmd Msg
performPlayerUpdate player =
    player |> playerUpdate |> Http.send PlayerUpdateHandler


playerDelete : Player -> Http.Request Player
playerDelete player =
    Http.request
        { method = "DELETE"
        , headers = []
        , url = "/api/players/" ++ (toString player.userId)
        , body = Http.emptyBody
        , expect = Http.expectStringResponse << always <| Ok player
        , timeout = Nothing
        , withCredentials = False
        }


performPlayerDelete : Player -> Cmd Msg
performPlayerDelete player =
    player |> playerDelete |> Http.send PlayerDeleteHandler



-- SUBSCRIPTIONS


subscriptions : Model -> Sub Msg
subscriptions model =
    Sub.none



-- VIEW


view : Model -> Html Msg
view model =
    div []
        [ viewSignUp model
        , viewPlayerStats model
        , viewPlayers model
        , createPlayerButton
        , fetchPlayersButton
        , viewErrors model
        ]



-- SIGN UP


viewSignUp : Model -> Html Msg
viewSignUp model =
    div [ class "row" ]
        [ Html.form [ class "sign-up form-group", onSubmit <| PlayerCreate model.newUsername ]
            [ h2 [] [ text "Sign Up" ]
            , label [ for "username-field" ] [ text "username: " ]
            , input
                [ class "username-field form-control"
                , placeholder "Enter username..."
                , type_ "text"
                , value model.newUsername
                , onInput Input
                ]
                []
            , label [ for "password" ] [ text "password: " ]
            , input
                [ class "password-field form-control"
                , placeholder "Enter password..."
                , type_ "password"
                , value ""
                ]
                []
            , button [ class "btn btn-success", type_ "submit" ] [ text "Sign Up" ]
            ]
        , button [ class "btn btn-info", onClick Clear ] [ text "Clear" ]
        ]



-- STATS


viewPlayerStats : Model -> Html Msg
viewPlayerStats model =
    div [ class "row" ]
        [ h2 [] [ text "Player Stats" ]
        , viewTotalScore model
        , viewHighScore model
        ]


viewTotalScore : Model -> Html Msg
viewTotalScore model =
    div []
        [ strong [] [ text "Total Score for All Players: " ]
        , span []
            [ model
                |> totalScore
                |> toString
                |> text
            ]
        ]


totalScore : Model -> Int
totalScore model =
    model.players
        |> List.map (\player -> player.score)
        |> List.sum


viewHighScore : Model -> Html Msg
viewHighScore model =
    div []
        [ strong [] [ text "High Score: " ]
        , span []
            [ model
                |> highScore
                |> Maybe.withDefault 0
                |> toString
                |> text
            ]
        ]


highScore : Model -> Maybe Int
highScore model =
    model.players
        |> List.map (\player -> player.score)
        |> List.maximum



-- VIEW PLAYERS


sortedPlayers : Model -> List Player
sortedPlayers model =
    model.players
        |> List.sortBy (\player -> player.userId)


viewPlayers : Model -> Html Msg
viewPlayers model =
    div [ class "row" ]
        [ h2 [] [ text "Player List" ]
        , ul [] (List.map viewPlayer (sortedPlayers model))
        ]


viewPlayer : Player -> Html Msg
viewPlayer player =
    li []
        [ span []
            [ strong [] [ text "Player ID: " ]
            , player.userId
                |> toString
                |> text
            ]
        , br [] []
        , span []
            [ strong [] [ text "Player Username: " ]
            , text player.username
            ]
        , br [] []
        , span []
            [ strong [] [ text "Player Score: " ]
            , player.score
                |> toString
                |> text
            ]
        , br [] []
        , span []
            [ button [ class "btn btn-info", PlayerUpdate player |> onClick ]
                [ text "Update Player" ]
            , button [ class "btn btn-danger", PlayerDelete player |> onClick ]
                [ text "Delete Player" ]
            ]
        ]



-- BUTTONS


createPlayerButton : Html Msg
createPlayerButton =
    button [ class "btn btn-primary", onClick <| PlayerCreate "default" ]
        [ text "Create Default Player" ]


fetchPlayersButton : Html Msg
fetchPlayersButton =
    button [ class "btn btn-primary", onClick PlayerFetch ]
        [ text "Refresh Player Data" ]



-- ERRORS


viewErrors : Model -> Html Msg
viewErrors model =
    case model.errors of
        "" ->
            div [] []

        _ ->
            div [ class "alert alert-danger" ] [ p [] [ text model.errors ] ]
```


