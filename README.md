# ddb config

This holds the default configurations for the [Discord Dice Bot](https://github.com/ttocsneb/discordDiceBot).  If you would like to add a default config for all servers, feel free to submit a pull request.

# Structure

The settings are separated into two parts.  The lookup table, and the config file.
The lookup table holds information for each config file.  The config file can be stored anywhere online as long as it accessible by a link, though it is preferred to be stored on this repo.

## ddbconf.json

ddbconf.json is the lookup table for the bot.  It holds two settings for each config:

- The description
- The url to the config

The json stucture looks like this.

```json
{
    "lookups": {
        "configname": {
            "desc": "Description for configname",
            "uri": "The url/(file uri if on a local machine)"
        },
        "configname2": ...
    }
}
```

## config file.json

The config file contains the actual configuration for the server.  It is split up into two parts: stats, and equations.  When the configuration is applied, the stats will be copied to the default stats. 

### Stats
 
Because stats are grouped, a stat is contained by a json object of the group name.  The value of a stat is simply the value of the stat which can be a number, or an equation. The equation can reference other stats or equations from the equation section.

Referencing a stat is simple: the name is surrounded by curly brackets.  If a variable doesn't exist, then an error will be thrown unless a default value is supplied.  A default value is placed after the name following a question mark.

Example:
```python
{group.name}
{name_without_group}
{variable.doesnt_exist?a_default_value}
{variable?{nested_variable}}
{variable?{nested_variable?{you_can_nest_a_lot}}}
```

### Equations

Equations are global to the server.  They serve as a simple way to perform calculations that everyone needs to do.  Equations do **not** have groups, but can have descriptions.

A description should be short and sweet as it is appended to the name whenever it's printed. The description is not required however.

The scheme for an equation setting can defined in two ways:

- The value is the equation
- The value is an object with a value key, and description key

Example:

```json
{
    "equation1": "The equation",
    "equation2": {
        "desc": "The description for the equation",
        "value": "The equation"
    }
}
```

### Config Scheme

Here is an example for a config file

```json
{
    "stats": {
        "group": {
            "stat": "5",
            "dep": "{group.stat}*2"
            "more": "{stat_without_group}+5"
        },
        null: {
            "stat_without_group": "1d6"
        }
    },
    "equations": {
        "eq1": "5*{0}",
        "calls_other_eqs": "eq1(1d6)",
        "uses_stats": "{group.more}/2",
        "has_desc": {
            "desc": "uses bar stat",
            "value": "{bar?5}+1d6"
        }
    }
}
```

# Equation Syntax

There are a lot of possibilities with equations.  At its core an equation is just a calculated number.  To calculate this number there are several primative functions that are available:

- `+-*/` These have two inputs and can be called in two ways: `a+b` or `+(a, b)`
- `^` the carrot puts an exponent on a number: `a^3 = a*a*a`
- `%` performs modulus (it gets the remainder from division): `15%10 = 5`
- `d` rolls a `b` sided dice `a` times: `adb`
- `round` rounds a single number to the nearest whole number: `round(5.5) = 6`
- `max min` gets the largest or smallest number of two: `max(1, 2) = 2`
- `floor ceil` rounds a number down, or up to a whole number: `ceil(1.1) = 2`
- `adv dis` rolls two `b` sided dice and gets the higher/lower roll: `adv(a)`
- `top bot` rolls `a` `b` sided dice and gets he highest/lowest `c` rolls: `top(a, , c)`
- `if` performs an excel style if statement: `if(test, on_true, on_false)` (a number is true if it is greater than 0)
- `< > = <> <= >=` comparison operators, compare two numbers and return true(1) or false(0): `a<b`
- `or` return true(1) if either input is true(>=1): `a or b` or `or(a, b)`
- `and` return true(1) if both inputs are true(>=1): `a and b` or `and(a, b)`

advanced functions can be created by creating equation objects.  These functions are just equations that allow for more advanced calculations.  The equation can have parameters which are defined by the parameter number starting at 0 surrounded by curly brackets:

```python
{0}: the first parameter
{1}: the second parameter
{2}: the third parameter

eq = {0} * 5

eq(2) = 2 * 5 = 10
eq(1d6) = (1d6) * 5
```

When an equation is called the values for its parameters are calculated first, so an equation like this `({0} + {0}^2)(1d6)` would not be the same as `1d6 + (1d6)^2`. The value for `1d6` would be calculated, say 5 then would be put into the function: `5 + 5^2 = 30`.

##Order of Operations

The order of operations is as follows calculated from high to low.

1. `< > = <> <= >= or and`
2. `+ -`
3. `* /`
4. `^ %`
5. `round max min floor ceil if` and all other custom equations
6. `adv dis top bot d`
7. `() {}`
