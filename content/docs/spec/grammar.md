+++
date = '2025-02-08T18:47:28+05:00'
draft = false
title = 'Alis Grammar'
+++
```ebnf
(*Note: the `.` special token is used to represent any non-newline character*)
(*Another Note: This grammar is not meant to generate parsers, only to be used
as documentation*)
whitespace = {space | "\t" | "\n"}-;
singleLineComment = "//", {? . ?}-, ? newline ?;
multiLineComment = "/*", {? . ?}-, "*/";
docComment = "///", {? . ?}-, ? newline ?;
identifier = ("_" | alphabet), {"_" | alphabet | decimalDigit};
shebang = "#!", {? . ?}, ? newline ?;

integerDecimal = {decimalDigit}-;
integerBinary = "0", ("b" | "B"), {"0" | "1"}-;
integerHex = "0", ("x" | "X"), {hexadecimalDigit}-;
integerScientific = (integerDecimal | literalFloat), "e", integerDecimal;
literalInt = integerDecimal | integerBinary | integerHex | integerScientific;

literalFloat = {decimalDigit}-, ".", {decimalDigit}-;
literalString = ('"', {? . ? | '\"'}, '"') |
				('"""', ? newline ?, {? . ? | '\"' | ? newline ?}, ? newline ?, '"""');
literalChar = "'", (("\", ? . ?) | ? . ? | ("\", "'")), "'";
literalBool = "true" | "false";

literalArray = "[", [{expr, ","}, expr], "]";

def = {attribute},
		 (fnDef | fnDef | varDef | structDef | unionDef | enumDef |
			aliasDef | templateDef | import);
globDef = ["ipub" | "pub"], def;

test = "utest", [literalString], statement;

moduleIdentifier = {identifier, "."}, identifier;
import = {moduleIdentifier, ["as", identifier], ","},
			 moduleIdentifier, ["as", identifier], ";";

statement = (expr | "break" | "continue" | mixinInit), ";" |
					block, [";"] | if | for | while | doWhile | switch | cComp | def;
block = "{", {statement}, "}";
if = "if", smExpr, statement, ["else", statement];
for = "for", "(", [identifier, ";"], expr, identifier, ";", expr, ")", statement;
while = "while", smExpr, statement;
doWhile = "do", statement, "while", smExpr, ";";
switch = "switch", smExpr, {"case", smExpr, block}, "case", "_", block;
mixinInit = "mixin", smExpr, "(", [{expr, ","}, expr], ")";

staticIf = "$if", smExpr, anyBlock, ["else", anyBlock];
staticFor = "$for", "(", [identifier, ";"], expr, identifier, ";", expr, ")",
					anyBlock;
staticSwitch = "$switch", smExpr, {"case", smExpr, anyBlock},
						 "case", "_", anyBlock;
cComp = {staticIf | staticFor | staticSwitch}-;

attribute = "#", smExpr;

tmParamList = "$(", [{tmParam, ","}, tmParam], ")", ["if", smExpr];
tmParam = ("$type", identifier | ("alias", identifier) | (smExpr, identifier)),
				[":", smExpr];

keyVal = identifier, "=", expr;
keyOptVal = identifier, ["=", expr];

fnDef = "fn", identifier, [tmParamList], paramList, "->",
			(smExpr, block | smExpr, ";");
methodDef = "fn", "[", smExpr, "]", identifier, [tmParamList], paramList, "->",
					(smExpr, block | smExpr, ";");
paramList = param | ("(", [{param, ","}, param], ")");
param = ([smExpr], keyOptVal) | smExpr;
fnAnon = "fn", paramList, "->", expr;

varDef = ("var" | "const"), smExpr, {keyOptVal, ","}, keyOptVal, ";";

aggMember = ["pub" | "ipub"], {attribute},
					(smExpr, [{keyOptVal, ","}, keyOptVal] |
					 "alias", identifier, "=", identifier), ";";
structDef = "struct", identifier, [tmParamList], "{",
					 {cComp | aggMember | mixinInit, ";"},
					 "}";
structExpr = "struct", "{", {cComp | aggMember | mixinInit, ";"}, "}";
structValExpr = "{", {cComp | keyVal | mixinInit, ";"}, "}";
unionDef = "union", identifier, [tmParamList], "{",
					{cComp | aggMember | mixinInit, ";"},
					"}";
unionExpr = "union", "{", {cComp | aggMember | mixinInit, ";"}, "}";
enumDef = "enum", smExpr, identifier, [tmParamList], ("=", expr, ";" |
		"{",
		{(cComp | keyOptVal | mixinInit), ","},
		[(cComp | keyOptVal | mixinInit)],
		"}"
		);

aliasDef = "alias", identifier, [tmParamList], "=", expr, ";";

anyBlock = "{", {? . ? | anyBlock} ,"}";

templateDef = "template", ["mixin"], tmParamList, anyBlock;

intrinsic = "$", identifier;

expr = smExpr, [block];
smExpr = ("(", {expr, ","}, expr, ")") | intrinsic |
			 literalInt | literalChar | literalBool | literalFloat |
			 literalString | fnAnon | structExpr | unionExpr | structValExpr |
			 literalArray | ("const", smExpr) |
			 (opPre, expr) | (expr, opPost) | (expr, opBin, expr) |
			 (expr, "(", [{expr, ","}, expr], ")") |
			 (expr, "[", [{expr, ","}, expr], "]");

opBin = "." | "??" | "!! "| "is" | "!is" | "*" | "/" | "%" | "+" | "-" | "<<" |
			">>" | "&" | "|" | "^" | ":" | "==" | "!=" | ">=" | "<=" | ">" | "<" |
			"&&" | "||" | "=" | "+=" | "-=" | "*=" | "/=" | "%=" |
			"&=" | "|=" | "^=" | "@=";

opPre = "@" | "is" | "!is" | "!" | "++" | "--" | "~" | "-";

opPost = "@" | "!" | "?" | "++" | "--" | "...";

module = [shebang], {globDef | test | mixinInit, ";"};
```
