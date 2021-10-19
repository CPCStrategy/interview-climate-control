# Tinuiti Backend Engineer Take Home Project Prompt

## The problem
In the wake of teleportation becoming the most common method of travel for humans across the world, a new wave of start ups are being created to address the challenges faced by the teleporting populous.

Your mission should you choose to accept it, is to build a GraphQL API using Ruby on Rails for a company called Climate Control.

Climate Control aims to solve the problem of figuring out which cities all over the world are currently have a comfortable climate, so that it's easy for people to decide which cities they would like to visit today.

## The challenge
We would like you to deliver a proof-of-concept for the back end of this project. Spending no more than **5 hours**: 
- create a Github/Gitlab repo for your project
- create the basic structure of a Ruby on Rails project that will provide a GraphQL API
- create a Readme file with instructions on setting up and running the project. Include all requirements to build/run from scratch.
- test the resulting API to ensure correctness against requirements 

We have provided the expected GraphQL schema. You may modify or extend the schema if you feel it better solves the problem.

We have checked in a minified list of cities to search against that provide the city identifier for the OpenWeather API. Normally, this would get loaded into a database lookup table but to reduce the complexity of proof of concept, all operations can be performed in-memory with no database or caching layer required.

We have also provided the tests that we expect to be able to execute against the running API to ensure correctness.

At the end of the project, please add a section to your Readme file that talks about any issues you encountered and directions you might have taken the project if you were not time constrained.  If you have any questions that aren't addressed by this readme, please reach out to the Hiring Manager.



## Project Requirements
- Ruby >= 3.0
- Rails >= 6.0
- [GraphQL Ruby](https://graphql-ruby.org/)
- Get weather data from [Open Weather Maps API](https://openweathermap.org/api)
    - Get an API key by [signing up](https://openweathermap.org/home/sign_up)

## Query and Mutation Descriptions
  - `hotOrNotCities`: Give back a list of current weather for all cities with the same name based on temp and arguments
    - `name`: The name of a city/cities
    - `hot`: Optional argument that indicated if you want hot or not hot cities returned. If left blank both with be returned
    - `feelsLike`: Weather the "feels like" temperature should be used to measure in place of the true temperature
  - `updateHotness`: Updates what the system defines as hot
    - `temp`: The temperature that defines what is hot
    - `units`: The temperature units to use for the system

## GraphQL Schema
At a minimum this is what your final GraphQL schema should look like
```GraphQL
  type Coordinates {
    lat: Float!
    lon: Float!
  }
  type City{
    coordinates: Coordinates!
    country: String!
    id: Int!
    name: String!
    sunrise: Datetime!
    sunset: Datetime!
    timeZone: Int!
  }
  type WeatherDescription {
    description: String!
    iconCode: String!
    id: Int!
    main: String!
  }
  type Wind {
    degrees: Int!
    gustSpeed: Float!
    speed: Float!
  }
  type Rain {
    volOneHour: Int!
    volThreeHours: Int!
  }
  type Snow {
    volOneHour: Int!
    volThreeHours: Int!
  }
  type OneDayWeather {
    cloudPer: Int!
    feelsLike: Float!
    groundLevel: Int
    hot: Boolean!
    humidity: Int!
    pressure: Int
    rain: Rain
    seaLevel: Int
    snow: Snow
    temp: Float!
    tempMax: Float!
    tempMin: Float!
    visibility: Int!
    weatherDescription: WeatherDescription
    wind: Wind
  }
  type CityWeatherOneDay {
    city: City!
    weather: OneDayWeather!
  }
  enum TempUnits {
    FAHRENHEIT
    CELSIUS
    KELVIN
  }
  type HotnessSetting {
    temp: Float!
    units: TempUnits!
  }
  type Query {
    hotOrNotCities(
      name: String!,
      hot Boolean,
      feelsLike: Boolean
      ): [CityWeatherOneDay!] = []
    hotnessSettings(): HotnessSetting!
  }
  type Mutation {
    updateHotnessSettings(
      temp: Float,
      units: TempUnits
    ): HotnessSetting!
  }
```

## Test Queries

### Basic Fields Check
Test to see that all the fields work with no hotness parameters
```GraphQL
query {
  hotOrNotCitiesWeather(name: "Portland") {
    city {
      coordinates {
        lat
        lon
      }
      country
      id
      name
      state
      sunrise
      sunset
      timeZone
    }
    weather {
      cloudPer
      feelsLike
      groundLevel
      hot
      humidity
      pressure
      rain {
        volOneHour
        volThreeHours
      }
      seaLevel
      snow {
        volOneHour
        volThreeHours
      }
      temp
      tempMax
      tempMin
      visibility
      weatherDescription {
        description
        iconCode
        id
        main
      }
      wind {
        speed
        degrees
        gustSpeed
      }
    }
  }
}

```

### Test Hot Argument
verify that cities in response when hot is set to true are not in list when hot is set to false
```GraphQL
  query {
    hotOrNotCitiesWeather(name: "Portland", hot: true) {
      city {
        id
      }
    }
  }
```

```GraphQL
  query {
    hotOrNotCitiesWeather(name: "Portland", hot: false) {
      city {
        id
      }
    }
  }
```

### Update Hotness Settings Based On Previous Query
We will pull the temp data from the previous query and use it to set the threshold to a value that falls between the temp and feels like for some cities
```GraphQL
  mutation {
    updateHotnessSettings(temp: 90.0, units: FAHRENHEIT) {
      temp
      units
    }
  }

```

### Test Feels Like Option
```GraphQL
  query {
    hotOrNotCitiesWeather(name: "Portland", feelsLike: true) {
      weather {
        feelsLike
        hot
        temp
      }
    }
  }

```
