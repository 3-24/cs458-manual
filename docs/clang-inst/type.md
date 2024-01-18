---
layout: default
title: Type Checking and Casting
parent: Clang Instrumentation
nav_order: 4
has_toc: true
---


### bool **isa\<*T*\>**(*Stmt \*stmt*)

Checks whether the input statement is type *T*. *T* can be statement or expression.

```c++
bool VisitStmt(Stmt *stmt) {
    if (isa<IfStmt>(stmt)){
        llvm::outs() << "You have reached if statement\n";
    }

    if (isa<ArraySubscriptExpr>(stmt)){
        llvm::outs() << "You have reached array subscript expression\n";
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
  if (a > 3) {
    a += 1;
  }

  if (a < 6){
    a -= 1;
  }
  else{
    a += 1;
  }

  int intarr[10][10];

  *intarr[1] = 42;
  return 0;
}
```
The output will be:
```
You have reached if statement
You have reached if statement
You have reached array subscript expression
```
</details>


### *T* \***dyn_cast\<*T*\>**(Stmt *stmt)

Casts a statement to an inherited statement type, such as *Stmt \** -> *IfStmt \**. This can also be applied to cast *Stmt* to underlying expression such as, *Stmt \** -> *MemberExpr \**. 

```c++
bool VisitStmt(Stmt *stmt) {
    if(isa<WhileStmt>(stmt)){
        Expr* cond = stmt->getCond();
    }
    return true;
}
```

If we try to execute the above code, there will be an compilation error because *Stm*t* does not have a member function `getCond`. Correct usage would be like below:
```c++
bool VisitStmt(Stmt *stmt) {
        if(isa<WhileStmt>(stmt)){
            auto whileStmt = dyn_cast<WhileStmt>(stmt);
            Expr * cond= whileStmt->getCond();
        }
        return true;
}
```

### *QualType* Expr::**getType**()

Obtains type of the given object. Use ```getAsString()``` function to obtain type in string form. Not applicable to ```Stmt``` objects. 

```c++
bool VisitStmt(Stmt *stmt) {
        if (isa<ArraySubscriptExpr>(stmt)){
            ArraySubscriptExpr* expr = dyn_cast<ArraySubscriptExpr>(stmt);
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
int main() {
  int intarr[10];
  intarr[1] = 42;
  return 0;
}
```
The output will be (```expr``` variable points to the whole expression ```intarr[1]``` which is integer, whereas ```expr->getBase()``` points to ```intarr```):
```
int
int *
```
</details>