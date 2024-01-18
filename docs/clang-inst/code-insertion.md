---
layout: default
title: Source Code Printing and Instrumentation
parent: Clang Instrumentation
nav_order: 2
---

## Printing the Source Code



### *string* Stmt::**getStmtClassName**()

Obtains a name of a statment class (e.g., IfStmt, WhileStmt, SwitchStmt).

```c++
bool VisitStmt(Stmt *stmt) {
    string stmtName = stmt->getStmtClassName();
    llvm::outs() << stmtName << '\n';
    return true;
}
```

<details markdown="block">
<summary>Example Usage</summary>
If we execute the above example on the code below:
```c
int main() {
    int a = 5;
    if (a > 3) {
        a += 1;
    }
    return 0;
}
```
The output will be:
```
CompoundStmt
DeclStmt
IntegerLiteral
IfStmt
BinaryOperator
ImplicitCastExpr
DeclRefExpr
IntegerLiteral
CompoundStmt
CompoundAssignOperator
DeclRefExpr
IntegerLiteral
ReturnStmt
IntegerLiteral
```
</details>

### *void* Stmt::**printPretty**(*raw_ostream &OS, PrinterHelper *Helper, const PrintingPolicy &Policy*)

Prints the statement/expression as it is.

```c++
bool VisitStmt(Stmt *stmt) {
    std::string str1;
    llvm::raw_string_ostream os(str1);
    stmt->printPretty(os, NULL, LangOpts);
    llvm::outs() << os.str();
    return true;
}
```

<details markdown="block">
<summary>Example Usage</summary>
If we execute the above example for the code below with a parameter `*stmt` that points to the `if` statement,
```c
int main() {
  int a = 5;
  int b = 4;
  if (a > b) {
    a += 1;
  }
  else {
    a -= 1;
  }
  return 0;
}
```

The output will be:
```
if (a > b) {
    a += 1;
} else {
    a -= 1;
}
```
</details>

## Get Access to Location

<h3 markdown="block">
*SourceLocation* Stmt::**getBeginLoc**()<br>
*SourceLocation* Stmt::**getEndLoc**()
</h3>

Get *SourceLocation* object that points to the beginning or the end of a given statement. To get the line and column information, use the following functions:
- SourceManager::**getExpansionLineNumber**(*SourceLocation loc*)
- SourceManager::**getExpansionColumnNumber**(*SourceLocation loc*)


```c++
bool VisitStmt(Stmt *stmt) {
    SourceLocation startLocation = stmt->getBeginLoc();
    unsigned int lineNum = m_sourceManager.getExpansionLineNumber(startLocation);
    unsigned int columnNum = m_sourceManager.getExpansionColumnNumber(startLocation);

    SourceLocation endLocation = stmt->getEndLoc();
    unsigned int lineNumEnd = m_sourceManager.getExpansionLineNumber(endLocation);
    unsigned int columnNumEnd = m_sourceManager.getExpansionColumnNumber(endLocation);

    llvm::outs() << lineNum << ", " << columnNum << "\n";
    llvm::outs() << lineNumEnd << ", " << columnNumEnd << "\n";
    return true;
}
```

<details markdown="block">
<summary>Example Usage</summary>
If we execute the above example on the code below, when ```*stmt``` points to the if statement/block:
```c
int main() {
  int a = 5;
  if (a > 3) {
    a += 1;
  }
  return 0;
}
```
The output will be:
```
3, 3
6, 3
```
</details>

<h3 markdown="block">
*SourceLocation* **getLParenLoc**()<br>
*SourceLocation* **getRParenLoc**()
</h3>

Return the location of the left and right paranthese if the statement has it. They can be used for *IfStmt*, *ForStmt* and so on, and not applicable to all types of statements.

```c++
bool VisitStmt(Stmt *stmt) {
    if(isa<IfStmt>(stmt)){
        auto ifStmt = dyn_cast<IfStmt>(stmt);
        SourceLocation startLocation = ifStmt->getLParenLoc();
        unsigned int lineNum = m_sourceManager.getExpansionLineNumber(startLocation);
        unsigned int columnNum = m_sourceManager.getExpansionColumnNumber(startLocation);

        SourceLocation endLocation = ifStmt->getRParenLoc();
        unsigned int lineNumEnd = m_sourceManager.getExpansionLineNumber(endLocation);
        unsigned int columnNumEnd = m_sourceManager.getExpansionColumnNumber(endLocation);
        llvm::outs() << lineNum << ", " << columnNum << "\n"
                     << lineNumEnd << ", " << columnNumEnd << "\n";
    }
    return true;
}
```

<details markdown="block">
<summary>Example Usage</summary>
if we execute the above example on the code below:
```c
int main() {
  int a = 5;
  if (
    a > 4
  ){
    a += 1;
  }
  return 0;
}
```
The output will be:
```
4, 6
6, 3
```
</details>

### *bool* SourceLocation::**isValid**()

Returns true if a given *SourceLocation* object is valid. 

```c++
bool VisitStmt(Stmt *stmt) {
    SourceLocation loc = stmt->getBeginLoc();
    if (loc.isValid())
        llvm::outs() << "Got valid SourceLocation object\n";
    else
        llvm::outs() << "Invalid SourceLocation\n";
    return true;
}
```
<details markdown="block">
<summary>Example Usage</summary>
If we execute the above example on the code below:
```c
int main() {
  int a = 5;
  int b = 10;
  return 0;
}
```
The output will be:
```
Got valid SourceLocation object
Got valid SourceLocation object
Got valid SourceLocation object
Got valid SourceLocation object
Got valid SourceLocation object
Got valid SourceLocation object
Got valid SourceLocation object
```
</details>

<h3 markdown="block">
*SourceLocation* Lexer::**findLocationAfterToken**(<br>
&nbsp;&nbsp;*SourceLocation Loc,*<br>
&nbsp;&nbsp;*tok::TokenKind TKind,*<br>
&nbsp;&nbsp;*const SourceManager &SM,*<br>
&nbsp;&nbsp;*const LangOptions &LangOpts,*<br>
&nbsp;&nbsp;*bool SkipTrailingWhitespaceAndNewLine*<br>
)
</h3>

Returns the location immediately after the specified token, if the token is right after the given location.  

```c++
bool VisitStmt(Stmt *stmt) {
    tok::TokenKind tokenKind = clang::tok::semi;
    unsigned int lineNum = m_sourceManager.getExpansionLineNumber(stmt->getBeginLoc());
    unsigned int columnNum = m_sourceManager.getExpansionColumnNumber(stmt->getBeginLoc());
    llvm::outs() << lineNum << ", " << columnNum << "\n";
    SourceLocation next_after_loc = clang::Lexer::findLocationAfterToken(
        stmt->getBeginLoc(), 
        tokenKind, 
        m_sourceManager, 
        LangOpts, 
        false
    );
    if (next_after_loc.isInvalid()){
      llvm::outs() << "Invalid SourceLocation\n";
    }
    else{
      lineNum = m_sourceManager.getExpansionLineNumber(next_after_loc);
      columnNum = m_sourceManager.getExpansionColumnNumber(next_after_loc);
      llvm::outs() << lineNum << ", " << columnNum << "\n\n\n";
    }
    return true;
}
```

<details markdown="block">
<summary>Example Usage</summary>

If we execute the above example on the code below:
```c
int main() {
  int a = 5;
  return 0;
}
```
The output will be:
```
1, 12 -> Beginning of the main function, the opening bracket of the function, next token is not semicolon, return Invalid SourceLocation
Invalid SourceLocation


2, 3 -> Beginning of the variable declaration, next token is not semicolon, return Invalid SourceLocation
Invalid SourceLocation


2, 11 -> The integer literal at line 2, column 11 (5), next token is semicolon, return the SourceLocation right after semicolon
2, 13


3, 3 -> Beginning of the return statement, next token is not semicolon, return Invalid SourceLocation
Invalid SourceLocation


3, 10 -> The integer literal at line 3, column 10 (0) next token is semicolon, return the SourceLocation right after semicolon
3, 12
```
</details>

## Inserting Source Code

Source code can be inserted at *SourceLocation* object.

<h3 markdown="block"> *bool* Rewriter::**InsertTextBefore**(*SourceLocation Loc, string text*)<br>
*bool* Rewriter::**InsertTextAfter**(*SourceLocation Loc, string text*)
</h3>

Modify a target source code.  

```c++
bool VisitStmt(Stmt *stmt) {
    if (isa<IfStmt>(stmt)) {
        IfStmt *ifStmt = dyn_cast<IfStmt>(stmt);
        TheRewriter.InsertTextBefore(
          ifStmt->getBeginLoc(),
          "  /*Before if statement*/\n"
        );
        TheRewriter.InsertTextAfter(
          ifStmt->getEndLoc().getLocWithOffset(1),
          "\n  /*After if statement*/\n"
        );
    }
    return true;
}
```

<details markdown="block">
<summary>Example Usage</summary>
If we execute the above example on the code below:
```c
int main() {
  int a = 5;
  if (a > 5){
    a += 1;
  }
  else{
    a -= 1;
  }
  return 0;
}
```
The output file will be:
```c
int main() {
  int a = 5;
  /*Before if statement*/
  if (a > 5){
    a += 1;
  }
  else{
    a -= 1;
  }
  /*After if statement*/

  return 0;
}
```
</details>


Why "After if statement" is inserted at the location obtained by **getLocWithOffset(1)**? It is because `getEndLoc()` function points to the token right before the end of the statement/expression. If we use it as it is in the above example, "After if statement" comment will actually be before the closing bracket of the else statement. Thus, we use the ```getLocWithOffset()``` function to obtain the desired location. ```getLocWithOffset()``` function moves the current location by the amount provided to the function. Negative argument will go backwards and positive arguments will go forward.  
