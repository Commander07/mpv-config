# Nix Flake + HM setup

I'm relativly new to Nix so this might not be optimal but feel free to reference and copy however much you want.

## mpv.nix
```nix
{ pkgs, ... }:

let
    guess-media-title = pkgs.mpvScripts.buildLua {
        pname = "guess-media-title";
        version = "0-unstable-commander-2024-10-29"; # i have modified `guess-media-title` to allow for changing the title format

        src = pkgs.fetchFromGitHub {
            owner = "Commander07";
            repo = "mpv-config";
            rev = "6508651105075f7c1a3267194eac16fc203515b1";
            hash = "sha256-JQ4w1pR9nLlydDi6IhSZ88fGs4itTTueDs0KlcChnnY=";
        };

        scriptPath = "./scripts/guess-media-title.lua";
        passthru.scriptName = "guess-media-title";
    };
in {
    home.packages = [
        pkgs.python312Packages.guessit # required for guess-media-title
    ];

    programs.mpv = {
        enable = true;
        package = (
            pkgs.mpv-unwrapped.wrapper { # if you are using x11 instead of wayland you should be able to get rid of this `package` definition just remember to move `scripts` if you do remove it
                scripts = with pkgs.mpvScripts; [
                    thumbfast
                    dynamic-crop
                    mpris
                    autosubsync-mpv
                ] ++ (with pkgs.unstable.mpvScripts; [
                    modernz
                    autosub
                ]) ++ [
                    guess-media-title
                    pkgs.unstable.mpvScripts.builtins.autoload
                    pkgs.unstable.mpvScripts.builtins.autodeint
                ];

                mpv = pkgs.mpv-unwrapped.override {
                    waylandSupport = true;
                };
            }
        );

        config = {
            resume-playback = true;
            save-position-on-quit = true;
            no-osc = true;
            osd-bar = false;
            image-display-duration = "inf";
            autocreate-playlist = "same";
        };

        profiles = {
            "loop-short" = {
                profile-cond = "duration < 30 and p['current-tracks/video/image'] == false";
                profile-restore = "copy";
                loop-file = true;
            };
        };

        scriptOpts = {
            autosubsync = { # the stable nix package automaticlly sets the sync provider to alass. unstable seems to set it to ffsubsync we are using the stable package you may want to use the unstable one
                overwrite_old_sub = true;
            };
            modernz = {
                showonpause = false;
                keeponpause = false;
                himetimeout = 1000;
                bottomhover = false;
                window_controls = true;
                ontop_button = false;
                info_button = false;
                seekbarfg_color = "#EE2E31";
                seekbarbg_color = "#40434E";
                hover_effect_color = "#ED272B";
                tooltips_for_disabled_elements = false;
                tooltip_hints = false;
                scalefullscreen = 0.5;
                scalewindowed = 0.6; # this can be quite small you may want to bump this up
            };
            guess-media-title = {
                episode_spec = "S%02d.E%02d";
                title_format = "%s %s \"%s\"";
                short_title_format = "%s %s";
            };
        };

        extraInput = ''
            MBTN_LEFT   cycle pause
        '';
    };
}
```
## flake.nix

```nix
{
    ...
    inputs = {
        ...
        nixpkgs.url = "github:NixOS/nixpkgs/nixos-24.11";
        nixpkgs-unstable.url = "github:NixOS/nixpkgs/nixos-unstable";
        ...
    };
    ...
    modules = [
        {
            nixpkgs.overlays = [
                ...
                (final: prev: {
                    unstable = import nixpkgs-unstable { # `import nixpkgs-unstable` here is a supposed anti pattern but i can't be bothered to change it. you may want to do a different solution entirely.
                        system = prev.system;
                        config = prev.config;
                    };
                })
                ...
            ];
        }
    ];
}
```
