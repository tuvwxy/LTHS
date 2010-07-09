High-level overview of directory tree

Top-level
	- src: Source codes.

Quick reference for the directory tree "src/"
	- build: Files related to building LTHS and related tools.
	- lths: Source codes for the game LTHS.
	- scrap: Test codes for trying trying out stuff. All codes in this 
	  directory should not be used directly in the main app.
	- tools: Other tools.

Quick reference for the directory tree "lths/"
	- ai: Codes for artificial intelligence?
	- animation: Codes for animation?
	- app: The "app" is the most basic level of the program. It is run on
	  startup, and dispatches to the game loop code.
		o locales: 
		o resources: Icons, cursors, localizations.
	- common: Files shared across all parts.
	- data: Data used in the game.
		- audio
		- models:
	- renderer: ???
	- test: 
	- third_party: Third party libraries that are used in LTHS.
