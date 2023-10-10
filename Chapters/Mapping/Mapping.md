## Mapping
	^(self ir instructionForPC: anInteger) sourceNode
	| initialPC pc |
	"generates the compiledMethod and optimize the ir.
	Removes the side-effect of optimizing the IR while looking for instruction,
	which results in incorrect found instruction"
	initialPC := self compiledMethod initialPC.
	"For a given PC, the actual instruction may start N bytes ahead."
	pc := aPC.
	[ pc >= initialPC ] whileTrue: [
		(self firstInstructionMatching: [:ir | ir bytecodeOffset = pc ])
				ifNotNil: [:it |^it].
		pc := pc - 1 ].
	^self "if we not found anything then this method is our target instruction"
	"Answer the program counter for the receiver's first bytecode."

	^ (self numLiterals + 1) * Smalltalk wordSize + 1
	| startpc |
	startpc := self method compiledMethod initialPC.
	self bytecodeIndex ifNil: [^startpc].
	^self bytecodeIndex + startpc - 1.
	| i |
	i := 5.
	[ i=0 ] whileFalse: [ i := i - 1 ].
42  ->  popIntoTemp: #i   ->  RBAssignmentNode(i := 5)
41  ->  goto: 2	          ->  RBMessageNode([ i = 0 ] whileFalse: [ i := i - 1 ])
43  ->  pushTemp: #i      ->  RBTemporaryNode(i)
44  ->  pushLiteral: 0    ->  RBLiteralValueNode(0)
45  ->  send: #=          ->  RBMessageNode(i = 0)
46  ->  if: true goto: 4
                 else: 3  ->  RBMessageNode([ i = 0 ] whileFalse: [ i := i - 1 ])
48  ->  pushTemp: #i      ->  RBTemporaryNode(i)
49  ->  pushLiteral: 1    ->  RBLiteralValueNode(1)
50  ->  send: #-          ->  RBMessageNode(i - 1)
51  ->  popIntoTemp: #i   ->  RBAssignmentNode(i := i - 1)
52  ->  goto: 2           ->  RBMessageNode([ i = 0 ] whileFalse: [ i := i - 1 ])
54  ->  returnReceiver    ->  helperMethod12  | i |
                                              i := 5.
                                              [ i = 0 ] whileFalse: [ i := i - 1 ]