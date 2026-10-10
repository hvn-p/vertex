# Vertex

A cooperative Survivor-like (up to 4 players, solo possible) where players build their
own spell from pieces. `vertex` is a code name. Design notes in progress live in
`docs/design.md`, a temporary file emptied into issues and code as decisions land.

All game terms, code and structure are in English.

## Glossary

| Term | Meaning |
|---|---|
| Entity | Anything that exists in the simulated world: avatar, enemy, projectile |
| Spell | What a player builds: an ordered list of Sequences. The order is free |
| Sequence | One stage of a Spell. Known kinds: `Form` (creates something: projectile, beam, zone) and `Impact` (what happens on contact or at the end) |
| Modifier | Plugs into a Sequence's socket and changes it: targeting, motion, size, speed, element |
| Piece | What a player finds during a run: a Sequence or a Modifier. Each Piece fits one kind of socket |
| Tree | How the game executes a Spell: each Sequence acts on what exists at its point of the list, so Sequences that create several things branch |
| Blueprint | A Spell saved in the profile as a model, built only from discovered Pieces. A run still has to find its Pieces |
| Library | The profile's record of discovered Pieces; undiscovered ones show hidden |

## Rules that shape the code

- Simulation and rendering are separate. Only the simulation is shared between players;
  cosmetics live in rendering only; the small shapes an Entity is drawn with are never
  simulated.
- The simulation steps at a fixed rate, every random draw comes from a seeded
  generator, a Spell is plain data, and a player acts on the simulation through inputs
  only.
- Optimize before capping. A limit goes where optimization can no longer keep up, never
  earlier.
- Visual style: Heptapode (`~/dotfiles/styles/heptapode/STYLE.md`). No gradient on an
  Entity: it is made of small shapes, each with its own opacity. Light is the exception.
  Changes to the style itself go to the style library, not here.
