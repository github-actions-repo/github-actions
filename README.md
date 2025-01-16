# DockerActionRecipe

A Docker container actions that handles input and output writes a job summary.

## Inputs

## `who-to-greet`

**Required** The name of the person to greet. Default `"World"`.

## Outputs

## `answer`

The answer to the ultimate question of life, the universe, and everything.

## Usage

```yaml
- uses: howsen82/DockerActionRecipe@v1.0
  with:
    # Required: the person to greet with the action
    # Default: World
    who-to-greet: ''
```

## Examples

### Simple example

```yaml
- uses: howsen82/DockerActionRecipe@v1.0
  with:
    who-to-greet: 'Steven Leong'
```

### Example that uses the output parameter

```yaml
- uses: howsen82/DockerActionRecipe@v1.0
  id: my-action
  with:
    who-to-greet: 'Steven Leong'

- name: Output the answer
  run: echo "The answer is '${{ steps.my-action.outputs.answer }}'"
```