# Starship Prompt Config
Documentation: [Configuration | Starship](https://starship.rs/config/)

## Create Config File

```bash
mkdir -p ~/.config && touch ~/.config/starship.toml
```

## My Personal Config File

```toml
format = """
$status \
$hostname\
$shlvl\
$kubernetes\
$directory\
$git_branch\
$git_commit\
$git_state\
$git_status\
$docker_context\
$package\
$nodejs\
$pulumi\
$vagrant\
$nix_shell\
$aws \
$cmd_duration\
$line_break\
$character"""

# Don't print a new line at the start of the prompt
add_newline = false

[aws]
format = '\[AWS: [$profile/($region)]($style)\]'
symbol = ''
style = 'bold white'

[character]
success_symbol = " [λ](grey)"
error_symbol = " [λ](bold red)"

[cmd_duration]
min_time = 1000

[directory]
truncation_length = 100
truncate_to_repo = false
style = " yellow"
format = "[$path]($style)[$read_only]($read_only_style) "

[git_branch]
symbol = ""
style = "bold white"
format = '[\($symbol$branch\)]($style) '

[git_status]
# I don't care about untracked files or that there's a stash present.
untracked = ""
format = '([\[$conflicted$deleted$renamed$modified$staged$behind\]]($style) )'
modified = '*'

[status]
disabled = false
format = '[\[$status - $common_meaning\]](green)'

#### Disabled modules ####

# add these back to format if you want them:
# $time\
# $hg_branch\
# $dart\
# $dotnet\
# $elixir\
# $elm\
# $erlang\
# $golang\
# $helm\
# $java\
# $julia\
# $kotlin\
# $nim\
# $ocaml\
# $php\
# $purescript\
# $swift\
# $zig\
# $memory_usage\
# $gcloud\
# $openstack\
# $crystal\
# $lua\
# $jobs\
# $battery\
[python]
disabled = true
[hg_branch]
disabled = true
[dart]
disabled = true
[dotnet]
disabled = true
[elixir]
disabled = true
[elm]
disabled = true
[erlang]
disabled = true
[golang]
disabled = true
[helm]
disabled = true
[java]
disabled = true
[julia]
disabled = true
[kotlin]
disabled = true
[nim]
disabled = true
[ocaml]
disabled = true
[php]
disabled = true
[purescript]
disabled = true
[swift]
disabled = true
[zig]
disabled = true
[memory_usage]
disabled = true
[gcloud]
disabled = true
[openstack]
disabled = true
[crystal]
disabled = true
[lua]
disabled = true
[jobs]
disabled = true
[battery]
disabled = true
[time]
disabled = true
```