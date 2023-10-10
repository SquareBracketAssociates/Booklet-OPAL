
ast := RBParser parseExpression: '1 + 2'.

"visit and annotated the AST for the closure analysis "
OCASTClosureAnalyzer new visitNode: ast.

"visit and annotated the AST for the var binding"
OCASTSemanticAnalyzer new
		scope: Object parseScope;
		visitNode: ast.