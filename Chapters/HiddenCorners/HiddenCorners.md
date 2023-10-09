## Hidden Corners### RoadmapSpecial stuff and super technical hidden stuff:- Doit compilation- Extra Bindings- Optimization  - inlining  - must be boolean explanation- Compiler plugins- Compiler options- Compilation Context- Environments / ProductionEnvironment### Special objectsSome  messages such  as the  ones of  Boolean are  not actually  sent. It  is then  impossible to  redefine suchmessages \(which ones\). This reason is that the compiler optimize them by inlining them. What is the problem### Supporting Inlined MessagesThere is an Opal modular extension to make sure that inlined messages can still be executed. \(more explanation needed\).```WHICH CLASS >> mustBeBoolean
	"Catches attempts to test truth of non-Booleans.  This message is sent from the VM.  The sending context is rewound to just before the jump causing this exception."
	^ Boolean mustBeBooleanDeOptimize
		ifTrue: [ self mustBeBooleanDeOptimizeIn: thisContext sender  ]
		ifFalse: [ self mustBeBooleanIn: thisContext sender ]```To enable it, you can just execute```Boolean mustBeBooleanDeOptimize: false.```and it is turned off.Note that `mustBeBooleanDeOptimizeIn:` is part of OpalCompiler, not the Kernel, so it does not take space there \(the only thing in the Kernel is the switch to turn it on or off\)In Opal, it is just this one method:```WHICH CLASS >> mustBeBooleanDeOptimizeIn: context
	"Permits to redefine methods inlined by compiler.
	Take the ast node corresponding to the mustBeBoolean error, compile it on
	the fly and executes it as a DoIt. Then resume the execution of the context."

	| sendNode methodNode method ret |

	"get the message send node that triggered mustBeBoolean"
	sendNode := context sourceNode sourceNodeForPC: context pc - 1.
	"Rewrite non-local returns to return to the correct context from send"
	RBParseTreeRewriter new
		replace: '^ ``@value' with: 'ThisContext home return: ``@value';
		executeTree: sendNode.
	"Build doit node to perform send unoptimized"
	methodNode := sendNode copy asDoitForContext: context.
	"Keep same compilation context as the sender node's"
	methodNode compilationContext: sendNode methodNode compilationContext copy.
	"Disable inlining so the message send will be unoptimized"
	methodNode compilationContext compilerOptions: #(- optionInlineIf optionInlineAndOr optionInlineWhile).
	"Generate the method"
	OCASTSemanticCleaner clean: methodNode.
	method := methodNode generate.
	"Execute the generated method with the pc still at the optimzized block so that the lookUp can read variables defined in the optimized block"
	ret := context receiver withArgs: {context} executeMethod: method.
  	"resume the context at the instruction following the send when returning from deoptimized code."
       context pc: sendNode irInstruction nextBytecodeOffsetAfterJump.
	^ret.```