Chess Logger
===
This project deals with reading images of a chessboard from the top down and creating a log of moves that have been made on either side. We used NI Vision Assistant for image processing and LabVIEW for vision acquisition and logic on the processed images. Our project was done for a machine vision class (ECE 4550) at the University of Virginia.

Usage
---
The project could theoretically be deployed to an NI MyRIO, but for testing we ran it on a laptop that was directly connected to a USB camera. The camera was attached to a copy stand that was placed above a large chessboard. For simplicity our chess pieces were custom cut acrylic that could be distinguished by pattern matching stages in Vision Assistant.

The front panel of the main program contains three buttons. The Calibrate button is used to figure out where in the video feed the chessboard is. The Initialize button creates the board layout without any knowledge of a previous state. If the Vision Assistant script is robust enough to not have any classification errors, any board layout can be used upon initialization; in reality, it is best to have all the pieces on the board at the start, as this will prevent false positives in classification from propagating to the board state. The Step button uses knowledge of the previous board state to generate a log string for a player's move (the log is displayed on the front panel).

A typical run of the program involves calibration first, and then initialization after that. Once these steps are done, one can move a chess piece belonging to the player whose turn it currently is (shown by the LEDs on the front panel) and then do a step. Then the other player can move and do a step. This process may continue indefinitely.

Software Description
---
A brief description of each LabVIEW file is provided.

**Vision Scripts**
* *chess_findobj.vascr* - used in the BoardStart and BoardStep states. Produces reports about shape matches for each piece type and each color (12 total).
* *chess_boardcal_hue.vascr* - used in Calibrate state. Produces a particle analysis report about green areas on the chess board (our board had green and white squares), which are then used to find where the corners of the board are.
* *chess_boardcal.vascr* - unused.
* *chess_findobj_maketemplate.vascr* - unused in LabVIEW, although it was used to create binary image templates that were then utilized in *chess_findobj.vascr*.

**TypeDefs**
* *chess_moveinfo_cluster.ctl* - cluster definition used in *step_moves.vi* to record where a piece moved from and to, as well as which piece got "replaced" by the moving piece.
* *chess_piecetype_enum.ctl* - enum for Pawn, Rook, Knight, Bishop, Queen, and King, as well as Blank.
* *chess_poschangeinfo_cluster.ctl* - cluster definition used in *board_stepcompare.vi* to record changes in the board state during BoardStep.
* *chess_posstate_cluster.ctl* - cluster definition that contains information about one location on a chessboard (i.e. which piece is present and which color it is).
* *chess_stateinfo_cluster.ctl* - cluster definition about state for the main FSM. Contains board layout, whose turn it is, a log string, board dimensions, and where we are in the FSM.
* *chess_states_enum.ctl* - enum for Wait, Calibrate, BoardStart, and BoardStep.
* *chess_teamcolor_enum.ctl* - enum for player color. Red or Blue, as well as None.

**Virtual Instruments**
* *best_score_ind.vi* - takes a shape report from *chess_findobj.vascr* and outputs the index of the shape match with the best score. Used in *fill_board*.
* *board_stepcompare.vi* - takes the current board layout, an array of shape reports, and the dimensions of the board. Iterates over the board as well as the shape reports to find differences between the previous state and the current state (i.e. if a location had a piece and is now blank, was blank and now has a piece, or had a piece of one color and now has a piece of a different color). These differences are logged in a *chess_poschangeinfo_cluster* and output in an array. Used in BoardStep state.
* *coord_to_str.vi* - takes integer coordinates of a grid location on the chessboard and creates its corresponding string. E.g. (2, 6) in grid coordinates translates to "C2". Used in *gen_logstring*.
* *coordinate_locator.vi* - takes a pixel location of a piece as well as the dimensions of the board, and outputs the grid coordinates corresponding to the piece.
* *fill_board.vi* - takes a shape report, a *chess_posstate_cluster*, a board layout, an expected number of pieces, and dimensions of the board. Finds the best match in the shape report and fills in the appropriate grid location on the board; if the location is already filled, then that match is skipped over. Process repeats until all matches have been scanned or the expected number of pieces have been found. Used in BoardStart state.
* *gen_logstring.vi* - takes a list of chess moves that have been recognized and creates log strings for them. E.g. "Blue Pawn moves from B2 to B4". Used in BoardStep state.
* *step_moves.vi* - takes a list of board differences from *board_stepcompare* and creates a list of chess moves, represented by an array of *chess_moveinfo_cluster*. Also updates the board layout with these moves. Used in BoardStep state.
* *chess_fsm.vi* - the main application program that glues together all the other subVIs. Has a state machine with Wait, Calibrate, BoardStart, and BoardStep states.

Future Work
---
We do not plan to develop the project further, but some possible ideas include:
* Notation - Have the log display formal chess notation.
* Rule validation - display an alert if a move has been made that is not valid (e.g. moving a pawn three spaces).
* AI integraton - have the computer suggest moves.
* Clock - much like competitive chess, have a running clock that shows how much time a player has left to act. The clock would get reset each time Step is pushed.
