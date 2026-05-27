# General

- The program should follow the XDG Base Directory specification for file locations

## Configuration files

- If no configuration file is specified at the command line, look in `$XDG_CONFIG_HOME` 
- Use Pydantic Settings Management for program configuration. 
    - Documentation is available here: `https://docs.pydantic.dev/latest/concepts/pydantic_settings/`

## Command Line Options

- Use `pydantic` to parse the command line options
    - Documentation is available here: `https://docs.pydantic.dev/latest/concepts/pydantic_settings/`
- The program should have a "--log-level" command line option that sets the logging level. Default to "WARN".
- The program should have a "--verbose" command line option that enables logging to the console. This should default to "False". If it is enabled show logs at the logging level set with "--log-level"
- The program should have a "--show-config" command line option that loads the configuration file (providing defaults for unspecified values), prints the configuration information to stdout in JSON format, and exits. The output should be structured so that it can be used as input to the "--config" command line argument
- The program should have a "--version" command that prints the current version of the program to stdout and exits. This command should also respect the "--format" specifier.
- The program should have a "--format" command line option that allows the user to select an output format. Typical values are: text, markdown, JSON, YAML
- The program should support a "--config" command line option that accepts the name of a configuration file and loads the program configuration from that file

## Logging

- The program should log to file by default. Log files should be located in the `$XDG_STATE_HOME` directory

