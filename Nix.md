# Nix Flake + HM setup

I'm relativly new to Nix so this might not be optimal but feel free to reference and copy however much you want.

## mpv.nix
```nix
{ pkgs, ... }:

let
    guess-media-title = pkgs.mpvScripts.buildLua { # the original does not exist on nixpkgs so i build it from my custom version instead of applying patches
        pname = "guess-media-title";
        version = "0-unstable-commander-2024-10-29";

        src = pkgs.fetchFromGitHub {
            owner = "Commander07";
            repo = "mpv-config";
            rev = "4e7766800a6aee5c4496eed0352184f89f7853ac";
            hash = "sha256-RDqye/3I0sE79X5ML/OGm+pX0osaetspXD4Ah++/s3U=";
        };

        scriptPath = "./scripts/guess-media-title.lua";
        passthru.scriptName = "guess-media-title.lua";
    };
in {
    home.packages = [
        pkgs.python312Packages.guessit # required for guess-media-title
    ];

    programs.mpv = {
        enable = true;
        package = (
            pkgs.mpv-unwrapped.wrapper {
                scripts = with pkgs.mpvScripts; [
                    thumbfast
                    dynamic-crop
                    mpris
                ] ++ (with pkgs.unstable.mpvScripts; [
                    modernz # does not exist on stable
                    autosub # does not exist on stable
                    (autosubsync-mpv.overrideAttrs (finalAttrs: prevAttrs: { # both stable and unstable are missing 'overwrite_old_sub' option
                        version = "0-unstable-2024-10-29";
                        src = pkgs.fetchFromGitHub {
                            owner = "joaquintorres";
                            repo = "autosubsync-mpv";
                            rev = "125ac13d1b84b3a64bb2e912225a8356c1c01364";
                            sha256 = "sha256-Xwu8WTB3p3YDTydfyidF/zpN6CyTQyZgQvGX/HAa9hw=";
                        };
                    }))
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
            osc = false;
            osd-bar = false;
            image-display-duration = "inf";
            autocreate-playlist = "same";
            vo = "gpu-next";
            hwdec = "vulkan-copy";
        };

        profiles = {
            "loop-short" = {
                profile-cond = "duration < 30 and p['current-tracks/video/image'] == false";
                profile-restore = "copy";
                loop-file = true;
            };
        };

        scriptOpts = {
            autosubsync = { # uses ffsubsync by default
                overwrite_old_sub = true;
            };
            modernz = {
                showonpause = false;
                keeponpause = false;
                hidetimeout = 1000;
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
                scalewindowed = 0.6;
            };
            guess-media-title = {
                invoke_python = false; # run guessit via 'guessit' instead of 'python -m guessit'
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
