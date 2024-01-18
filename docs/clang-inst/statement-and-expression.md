---
layout: default
title: Accessing Elements of Statements and Expressions
parent: Clang Instrumentation
nav_order: 3
has_toc: true
---

## Conditional Statements

### *Expr* \***getCond()**

Obtains the conditional expression in statements such as *If*, *For* and etc. 

```c++
bool VisitStmt(Stmt *stmt) {
    if(isa<WhileStmt>(stmt)){
        auto whileStmt = dyn_cast<WhileStmt>(stmt);
        Expr * cond= whileStmt->getCond();

        std::string str1;
        llvm::raw_string_ostream os(str1);
        cond->printPretty(os, NULL, LangOpts);
        llvm::outs() << os.str() << "\n";
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
  while (a < 10){
    a += 1;
  }
  return 0;
}
```
The output will be:
```
a < 10
```
</details>

<h3 markdown="block">
*Stmt* \*IfStmt::**getThen**()<br>
*Stmt* \*IfStmt::**getElse**()
</h3>

Obtains then and else part of the *if* statement. 

```c++
bool VisitStmt(Stmt *stmt) {
        if(isa<IfStmt>(stmt)){
            auto ifStmt = dyn_cast<IfStmt>(stmt);
            Expr* cond = ifStmt->getCond();
            Stmt* then = ifStmt->getThen();
            Stmt* elseS = ifStmt->getElse();

            std::string cond_str;
            llvm::raw_string_ostream os(cond_str);
            cond->printPretty(os, NULL, LangOpts);
            llvm::outs() << os.str() << "\n\n";

            std::string then_str;
            llvm::raw_string_ostream os1(then_str);
            then->printPretty(os1, NULL, LangOpts);
            llvm::outs() << os1.str() << "\n\n";

            std::string else_str;
            llvm::raw_string_ostream os2(else_str);
            elseS->printPretty(os2, NULL, LangOpts);
            llvm::outs() << os2.str() << "\n\n";
        }
        return true;
}
```
<details markdown="block">
<summary>Example Usage</summary>
If we execute the above example on the code below and apply ```printPretty``` to them:
```c
int main() {
  int a = 5;
  int b = 6;
  if (a > 4){
    a += 1;
    b -= 1;
  }else{
    a -= 1;
    b += 1;
  }
  return 0;
}
```
The output will be:
```
a > 4

{
    a += 1;
    b -= 1;
}


{
    a -= 1;
    b += 1;
}
```
</details>

### *Stmt*\* SwitchCase::**getSubStmt**()
Gets the statement block that corresponds to the switch case. 


```c++
bool VisitStmt(Stmt *stmt) {
    if (isa<SwitchCase>(stmt)) {
        SwitchCase *switchCase = cast<SwitchCase>(stmt);
        auto subStmtm = switchCase->getSubStmt();
        
        std::string case_block_str;
        llvm::raw_string_ostream os(case_block_str);
        subStmtm->printPretty(os, NULL, LangOpts);
        llvm::outs() << os.str() << "\n\n";
    }
    return true;
}
```
<details markdown="block">
<summary>Example Usage</summary>
If we execute the above example on the code below:
```c
int main() {
  int a = 4;
  
  switch(a){
    case 1: a += 1;
    case 2: a *= 2;
    case 3: a -= 3;
    default: a = 0;
  };

  return 0;
}
```
The output will be:
```
a += 1

a *= 2

a -= 3

a = 0
```
</details>

<h3 markdown="block">
*SwitchCase* \*SwitchStmt::**getSwitchCaseList**()<br>
*SwitchCase* \*SwitchCase::**getNextSwitchCase**()
</h3>

Iterates through the cases of the switch statement. 

```c++
bool VisitStmt(Stmt *stmt) {
    if (isa<SwitchStmt>(stmt)) {
        auto switchStmt = cast<SwitchStmt>(stmt);
        auto switchCase = switchStmt->getSwitchCaseList();
        
        while (switchCase){
            std::string case_str;
            llvm::raw_string_ostream os(case_str);
            switchCase->printPretty(os, NULL, LangOpts);
            llvm::outs() << os.str() << "\n\n";
            switchCase = switchCase->getNextSwitchCase();
        }

    }
    return true;
}
```

<details markdown="block">
<summary>Example Usage</summary>
If we execute the above example on the code below:
```c
int main() {
  int a = 4;
  
  switch(a){
    case 1: a += 1;
    case 2: a *= 2;
    case 3: a -= 3;
    default: a = 0;
  };

  return 0;
}
```
The output will be:
```
default:
a = 0;


case 3:
a -= 3;


case 2:
a *= 2;


case 1:
a += 1;
```
</details>

## Loop Statements

### *Stmt* \*Forstmt::**getBody**()

Obtains the body of a given statement. Can be applied to loops and functions.  

```c++
bool VisitStmt(Stmt *stmt) {
    if (isa<ForStmt>(stmt)){
        ForStmt * forStmt = dyn_cast<ForStmt>(stmt);
        Stmt* body = forStmt->getBody();

        std::string body_str;
        llvm::raw_string_ostream os(body_str);
        body->printPretty(os, NULL, LangOpts);
        llvm::outs() << os.str() << "\n\n";
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
  int b = 10;
  int i;
  for (i = 0; i < 10; i++){
    a += 1;
    b += 2;
  }
  
  return 0;
}
```
The output will be:
```
{
    a += 1;
    b += 2;
}
```
</details>

## Structural Expressions

Structural expressions includes accessing elements inside pointer, struct, and array.

<h3 markdown="block">
*Expr* \*MemberExpr::**getBase**()<br>
*Expr* \*ArraySubscriptExpr::**getBase**()
</h3>

Obtains base expression if the target expression is *MemberExpr*, *ArraySubscriptExpr*. 

```c++
bool VisitStmt(Stmt *stmt) {
    if (isa<MemberExpr>(stmt)){
        MemberExpr* expr = dyn_cast<MemberExpr>(stmt);
        auto t = expr->getType().getAsString();
        auto t_base = expr->getBase()->getType().getAsString();

        llvm::outs() << t << "\n";
        llvm::outs() << t_base << "\n";
    }
    return true;
}
```

<details markdown="block">
<summary>Example Usage</summary>
If we execute the above example on the code below:
```c
struct a {
  int b;
};

int main() {
  struct a * aptr;
  aptr->b = 5;
  return 0;
}
```
The output will be (```expr->getBase()``` points to ```aptr```):
```
int
struct a *
```
</details>


### *DeclarationNameInfo* \*MemberExpr::**getMemberNameInfo**()

Obtains information about the member variable. 

```c++
bool VisitStmt(Stmt *stmt) {
        if (isa<MemberExpr>(stmt)){
            MemberExpr * memexpr = dyn_cast<MemberExpr>(stmt);
            auto memberName = memexpr->getMemberNameInfo();
            llvm::outs() << memberName << "\n";
        }
        return true;
}
```

<details markdown="block">
<summary>Example Usage</summary>
If we execute the above example on the code below:
```c
struct a {
  int myCustomVariable;
};

int main() {
  struct a aptr1 = {.myCustomVariable=3};
  aptr1.myCustomVariable = 5;
  return 0;
}
```
The output will be:
```
myCustomVariable
```
</details>


### *bool* MemberExpr::**isArrow**()

To learn whether the member access is arrow or dot access. 

```c++
bool VisitStmt(Stmt *stmt) {
    if (isa<MemberExpr>(stmt)){
        MemberExpr * memexpr = dyn_cast<MemberExpr>(stmt);
        if(memexpr->isArrow()){
            llvm::outs() << "We got the memeber access with arrow\n";
        }
        else{
            llvm::outs() << "We got the member access with dot\n";
        }
    }
    return true;
}
```

<details markdown="block">
<summary>Example Usage</summary>
If we execute the above example on the code below:
```c
struct a {
  int myCustomVariable;
};

int main() {

  struct a aptr1 = {.myCustomVariable=3};
  aptr1.myCustomVariable = 5;

  struct a* aptr = &aptr1;
  aptr->myCustomVariable = 5;
  return 0;
}
```
The output will be:
```
We got the member access with dot
We got the memeber access with arrow
```
</details>

### *Expr* \***getSubExpr**()

This base expression used in the dereferencing expression, such as ```*pointer``` -> ```getSubExpr``` will give you ```pointer```. 
  
```c++
bool VisitStmt(Stmt *stmt) {
    if (isa<UnaryOperator>(stmt)) {
        UnaryOperator * unaryexpr = dyn_cast<UnaryOperator>(stmt);

        if (unaryexpr->getOpcode() == UnaryOperatorKind::UO_Deref) {
            
            std::string expr_str;
            llvm::raw_string_ostream os1(expr_str);
            expr_str->printPretty(os1, NULL, LangOpts);
            llvm::outs() << os1.str() << "\n\n";

            Expr * subexpr = unaryexpr->getSubExpr();
            std::string pointer_str;
            llvm::raw_string_ostream os(pointer_str);
            subexpr->printPretty(os, NULL, LangOpts);
            llvm::outs() << os.str() << "\n\n";
        }
    }
    return true;
}
```

<details markdown="block">
<summary>Example Usage</summary>
if we execute the above example on the code below:
```c

int main() {
  int a = 4;
  int* pointer = &a;
  *pointer = 5;
  return 0;
}
```
The output will be:
```
*pointer

pointer
```
</details>

## Functions

<h3 markdown="block">
*Stmt* \*FunctionDecl::**getBody**()<br>
*bool* FunctionDecl::**hasBody**()<br>
*string* FunctionDecl::**getName**()
</h3>

These functions can be used to obtain information about the given function. 

```c++
bool VisitFunctionDecl(FunctionDecl *f) {
    if (f->hasBody()) {
        Stmt *FuncBody = f->getBody();
        std::string body_str;
        llvm::raw_string_ostream os(body_str);
        FuncBody->printPretty(os, NULL, LangOpts);
        llvm::outs() << f->getName() << "\n";
        llvm::outs() << os.str() << "\n\n";
    }
    
    return true;
}
```

<details markdown="block">
<summary>Example Usage</summary>
If we execute the above example on the code below:
```c
int customFunc(){
  int b = 1;
  int a = 2;
  return a + b;
}

int main() {
  int a = 5;
  int b = 10;
  return 0;
}
```
The output will be:
```
customFunc
{
    int b = 1;
    int a = 2;
    return a + b;
}


main
{
    int a = 5;
    int b = 10;
    return 0;
}
```
</details>

### *bool* FunctionDecl::**isMain**()

Checks whether the current function is main. 

```c++
bool VisitFunctionDecl(FunctionDecl *f) {
    if (f->isMain()) {
        llvm::outs() << "Got the main function" << "\n";
    }
    else {
        llvm::outs() << "Not the main function\n";
    }
    
    return true;
}
```
<details markdown="block">
<summary>Example Usage</summary>
If we execute the above example on the code below:
```c
int customFunc(){
  int b = 1;
  int a = 2;
  return a + b;
}

int main() {
  int a = 5;
  int b = 10;
  return 0;
}
```
The output will be:
```
Not the main function
Got the main function
```
</details>