# IMSDb Movie Script Parser
A python parser for movie scripts.
This project was conducted during the class "Object Oriented Programming in Python", given by Davide Picca and assisted by Coline Métrailler during the spring semester 2020 at the University of Lausanne (UNIL).

We used a database of 1119 english-speaking movie scripts in plain text format (.txt), retrieved from the [Internet Movie Script Database](https://www.imsdb.com/). 

The parsed JSON files generated by the code can be found in the `parsed_scripts` folder of this repository.

## Dependencies
- Python 3
- [dataclasses_json](https://pypi.org/project/dataclasses-json/)

## Usage
Create an instance of the `ScriptParser` class. Call the `parse()` method with a string corresponding to an entire movie script. It outputs a `Movie` instance.

```python
from ScriptParser import ScriptParser

parser = ScriptParser()

parsed_movie = parser.parse(my_movie_string)
```

The full pipeline that we used to generate the JSON files using the parser can be found in the `main.py` script.

## Data
The parser returns data that is structured as follows. They can be converted into JSON format using the `to_json()` method, thanks to the `dataclasses_json` library.

### Movie
This class represents a parsed movie. It regroups metadata, and a list of all the characters speaking in the script.

| Attribute     | type              | Description  |
| ------------- |-------------------| ------------ |
| `Title`         | `str`             | the title of the movie |
| `Author`        | `list(str)`       | a list of the script's writers |
| `Genre`         | `list(str)`       | a list of the movie's genres |
| `Characters`    | `list(Character)` | a list of the characters speaking in this movie

### Character
This class represents one of the movie's characters. It is contained in a `Movie`'s `Characters` list.

A `get_replies()` method allows access to the list of replies.

| Attribute     | type              | Description  |
| ------------- |-------------------| ------------ |
| `Character`     | `str`             | the name of the character |
| `Replies`       | `list(Reply)`     | a list of the character's replies in the script |

### Reply
This class represents one reply spoken by a character. It is contained in a `Character`'s `Replies` list.

A `get_start()` and a `get_end()` method allows access to the corresponding attribute value.

| Attribute  | type  | Description  |
| ---------- |-------| ------------ |
| `Reply`      | `str` | the text of this reply |
| `Didascalie` | `str` | the text of the didascalie. If there is no didascalie, the default value is an empty string. |
| `Start`      | `int` | start position in number of characters from the start of the script |
| `End`        | `int` | end position in number of characters from the start of the script |

## Parameters
You can overwrite or change certain attributes of a `ScriptParser`, depending on your needs.
| Attribute           | type  | Description  |
| ------------------- |-------| ------------ |
| `minimum_replies`     | `int` | The minimal amount of replies under which a character is rejected. The default value is 1. This discards all characters with no replies. With the value adjusted to 2, most false-positives will be eliminated, but some minor characters along with them. |
| `minimum_characters`  | `int` | The minimal amount of characters under which a movie is rejected. The default value is 5. Further explanations may be found in the [Special scripts](#special-scripts) section.|
| `character_blacklist` | `list(str)` | A list of strings. If any of the strings in the list is contained in a character name, the character will be rejected. |
| `reply_blacklist`     | `list(str)` | A list of strings used to filter replies. If any of the strings are found in the reply (case sensitive), the reply will be rejected |

`character_blacklist` and `reply_blacklist` already contain some black listed words (e.g. `INT.` or `NIGHT`). You can completely overwrite the list, or append additional words:

```python
parser = ScriptParser()

# add words to the blacklist
parser.character_blacklist += ["NEW", "WORDS"]

# overwrite the blacklist
parser.character_blacklist = ["ENTIRELY", "NEW", "WORDS"]
```
## Special scripts
Special scripts are determined by checking if a parsed script has less than `minimum_characters` characters. The script will then be sent to the `_special_characters()` method which will attempt to identify one of three special cases:

1. The entire text of the script is contained on a single line.
2. Character names are not capitalized and/or are placed on the same line as their replies, followed by a colon.
3. Character names are not capitalized and are not followed by a colon.

A specifically modified text is then returned and sent to the `_get_characters()` method to be parsed a second time.

## Error
If the script does not fit any of these cases or still has less than `minimum_characters` characters after being parsed a second time, you will get the following error message: "The script couldn't be parsed."
There are a couple things you can do to fix it.

1. Reformat the script so that the `ScriptParser` can parse it correctly. 
Character names should be in capital letters and finish with a line break. Replies should start on the line right after the character name and finish with a punctuation symbol followed by a line break. Whitespace may be placed before and after the character names in order to improve readability, but can NOT be placed after the final punctuation symbol in the reply.

Example:

                            SCULLY
               I know you're bored in this
               assignment, but unconventional
               thinking is only going to get you into
               trouble now.

                            MULDER
               How's that?

                            SCULLY
               You've got to quit looking for what
               isn't there. They've closed the
               X-Files. There's procedure to be
               followed here. Protocol.

                            MULDER
               What do you say we call in a bomb
               threat for Houston. I think it's free
               beer night at the Astrodome.

2. You can also add a new regex with a corresponding `if len(matches) > fail:` condition to the `_special_characters()` method in the `ScriptParser` class. This is an interesting option if you have many scripts with the same syntax. 
    - Add your new regex in `_special_characters()` in `ScriptParser.py`: it will identify scripts that have a particular syntax.
    - Return a modified script that can be parsed by the `_get_characters()` method. DISCLAIMER: This modified script must replicate all whitespace present in the original script in order for the positional methods `get_start()` and `get_end()` from the `Reply` class to work correctly, respecting the whitespace conditions mentioned above.

## Contributors
- Melinda Femminis
- Florian Rieder
- Andres Stadelmann
