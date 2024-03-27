# Modularize video repo

## module with enable

```nix
# module1.nix

{ pkgs, lib, config, ... }: {

  options = {
    module1.enable = 
      lib.mkEnableOption "enables module1";
  };

  config = lib.mkIf config.module1.enable {
    option1 = 5;
    option2 = true;
  };

```

## flake

```nix
{
  description = "flake";

  # ...

  outputs = { nixpkgs, ... }@inputs: {
    nixosConfigurations.default = nixpkgs.lib.nixosSystem {
      specialArgs = { inherit inputs; };
      modules = [
        ./hosts/host1/configuration.nix
        ./nixosModules
      ];
    };

    # more configurations like this
  };

}
```

## standalone home-manager flake

```nix
{
  description = "flake";

  # ...

  outputs = { nixpkgs, ... }@inputs: let
      system = "x86_64-linux";
      pkgs = nixpkgs.legacyPackages.${system};
    in {
      homeConfigurations."username" = home-manager.lib.homeManagerConfiguration {
        inherit pkgs;

        modules = [ 
          ./home.nix 
          ./homeManagerModules
        ];

      };
  };

}
```

## home-manager module flake

```nix
{
  description = "flake";

  # ...

  outputs = { nixpkgs, ... }@inputs: {

    nixosConfigurations.default = nixpkgs.lib.nixosSystem {
      specialArgs = { inherit inputs; };
      modules = [
        ./hosts/host1/configuration.nix
        ./nixosModules
      ];
    };

    homeManagerModules.default = ./homeManagerModules;

  };

}
```

## home-manager module

```nix
# home-manager.nix

{ inputs, ... }: {

  # may look a bit different
  home-manager."username" = {
    extraSpecialArgs = { inherit inputs; };
    users = {
      "username" = import ./home.nix;`)}
      modules = [
        ./home.nix
        inputs.self.outputs.homeManagerModules.default
      ];
    };
  };

}
```
