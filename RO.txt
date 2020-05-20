import numpy as np
def initialise (x,y):
    L=[]
    for i in range(y):
        l=[]
        print("entrez l'equation N° " +str(i+1)+" : ")
        for j in range(x):
            l.append(input_modified("entrez le coefficient de x"+str(j+1)+": "))
        l.append(input_modified(" <= "))
        L.append(l)
    l=[]
    print("max Z = ")
    for j in range(x):
        l.append(input_modified("entrez le coefficient de x"+str(j+1)+": "))
    L.append(l)
    return L

def input_modified(msg):
    res = input(msg)
    if("/" in res):
        return float(res[0])/float(res[2])
    else:
        return int(res)

def rendre_dual(primal,x,y):
    dual =[]
    for i in range(x):
        d=[]
        for j in range(y):
            d.append(primal[j][i])
        d.append(primal[y][i])
        dual.append(d)
    d=[]
    for j in range(y):
        d.append(primal[j][x])
    dual.append(d)
    return dual

def getsol(x):
    res=[]
    print("entez la solution proposée")
    for i in range(x):
        res.append(input_modified("entrez la valeur de x"+str(i)+": "))
    return res

def commentaire(sol,primal,x,y):
    l=[]
    r=[]
    for i in range(x):
        if (sol[i]!=0):
            l.append(i)
        else:
            r.append(i)
    return [l,r]

def afficher_commentaire(contr_non_sat):
    if(len(contr_non_sat)>0):
        if(len(contr_non_sat)==1):
            msg = "est"
            msg2 = "la contraite "
        else:
            msg = "sont"
            msg2 = "les contraintes "
        print("on a ",end="")
        for i in range(len(contr_non_sat)):
            print("x"+str(contr_non_sat[i]+1),end=" ")
        print(msg+" > 0 donc "+msg2,end="")

        for i in range(len(contr_non_sat)):
            print(str(contr_non_sat[i]+1),end=", ")
        print("du problème dual "+msg+" saturée")

def afficher_commentaire2(primal,x,y,sol_propose):
    yi_nulles=[]
    for i in range(y):
        tot=0
        for j in range(x):
            tot += sol_propose[j]*primal[i][j]
        if(round(abs(tot -primal[i][x]),3)!=0):
            yi_nulles.append(i)
    if(len(yi_nulles)==1):
        msg = "l'equation"
        e = "n'est pas saturée"
    else:
        msg = "les equations"
        e = "ne sont pas saturées"
    print("on a "+msg,end=" ")
    for j in range(len(yi_nulles)):
        print(str(yi_nulles[j]+1),end =", ")
    print(e+" et alors",end =" ")
    for i in range(len(yi_nulles)):
        print("y"+str(yi_nulles[i]+1)+"= 0",end=", ")
    print("")
    return yi_nulles

def afficher_equation_prblm(primal,x,y,var):
    print("max Z = ",end="")
    if (primal[y][0]>0):
        print(str(primal[y][0])+var+str(1),end="\t")
    elif primal[y][0]==0:
        print("",end="\t")
    else:
        print("-\t"+str(abs(primal[y][0]))+var+str(1),end="\t")
    for i in range(1,x):
        if (primal[y][i]>0):
            print("+\t"+str(primal[y][i])+var+str(i+1),end="\t")
        elif primal[y][i]==0:
            print(" \t",end="\t")
        else:
            print("-\t"+str(abs(primal[y][i]))+var+str(i+1),end="\t")
    print("")
    for i in range(y):
        print(str(primal[i][0])+var+str(1),end="\t")
        for j in range(1,x):
            if (primal[i][j]>0):
                print("+\t"+str(primal[i][j])+var+str(j+1),end="\t")
            elif primal[i][j]==0:
                print("\t",end="\t")
            else:
                print("-\t"+str(-primal[i][j])+var+str(j+1),end="\t")
        if var == "y":
            print (">= "+str(primal[i][x]))
        else:
            print ("<= "+str(primal[i][x]))


def equation_obtenue(dual,x,y,yi_nulles,contr_non_sat):
    L=[]
    for e in contr_non_sat:
        l=[]
        for j in range(y):
            if j in yi_nulles:
                val = 0
            else:
                val = dual[e][j]
            l.append(val)
        l.append(dual[e][y])
        L.append(l)
    return L

def afficher_equation(equation ,var):
    for i in range(len(equation)):
        print(str(equation[i][0])+var+str(1),end="\t")
        for j in range(1,len(equation[0])-1):
            if (equation[i][j]>0):
                print("+\t"+str(equation[i][j])+var+str(j+1),end="\t")

            if (equation[i][j]<0):
                print("-\t"+str(-equation[i][j])+var+str(j+1),end="\t")
        print ("<= "+str(equation[i][len(equation[0])-1]))

def solution_equation(equation,yi_nulles):
    res =[]
    A=[]
    B=[]
    for e in equation:
        A.append(e[:-1])
        B.append(e[-1])
    for x in range(len(equation)):
        extra=0
        for i in yi_nulles:
            del(A[x][i-extra])
            extra+=1
    A=np.array(A)
    B=np.array(B)
    res=np.linalg.solve(A,B)
    return res

def afficher_solution(sol,yi_nulles):
    index=0
    L=[]
    if 0 in yi_nulles:
        print("(y1 = "+str(0),end=", ")
        L.append(0)
    else:
        print("(y1 = "+str(round(sol[index],3)),end=", ")
        L.append(round(sol[index],3))
        index=index+1

    for i in range(1,len(sol)+len(yi_nulles)-1):
        if i in yi_nulles:
            print("y"+str(i+1)+" = "+str(0),end=", ")
            L.append(0)
        else:
            print("y"+str(i+1)+" = "+str(round(sol[index],3)),end=", ")
            L.append(round(sol[index],3))
            index=index+1

    if len(sol)+len(yi_nulles)-1 in yi_nulles:
        print("y"+str(len(sol)+len(yi_nulles))+" = "+str(0)+")")
        L.append(0)
    else:
        print("y"+str(len(sol)+len(yi_nulles))+" = "+str(round(sol[index],3))+")")
        L.append(round(sol[index],3))
    return L

def checkmax(primal,dual,res,sol_propose):
    Z=0
    W=0
    for i in range(len(primal[-1])):
        Z+=primal[-1][i]*sol_propose[i]
    for i in range(len(dual[-1])):
        W+=dual[-1][i]  *res[i]
    if(W==Z):
        print("************************************")
        print("******** MAX (Z) = MIN (W) *********")
        print("************************************")
        print("***SOLUTION PROPOSEE EST OPTIMALE***")
        print("************************************")
    else:
        print("************************************")
        print("******** MAX (Z) != MIN (W) ********")
        print("************************************")
        print("***SOLUTION PROPOSEE NON OPTIMALE***")
        print("************************************")

def check_rest(primal,dual,check, res,sol_propose):
    eq=0
    for i in range(len(check)):
        tot=0
        for j in range(len(dual[0])-1):
            tot+=dual[check[i]][j]*res[j]

        if tot>=dual[check[i]][len(dual[0])-1]:
            eq+=1
            print("V- equation "+str(check[i]+1)+" est vérifiée par la solution\n")
        else :
            print("X- equation "+str(check[i]+1)+" n'est pas vérifie par la solution\n")
    if eq==len(check):
        print("************************************")
        print("**************** OK ****************")
        print("************************************\n")
        checkmax(primal,dual,res,sol_propose)
    else:

        print("************************************")
        print("***SOLUTION PROPOSEE NON OPTIMALE***")
        print("************************************")



x = input_modified("entrez le nombre de variables: ")
y = input_modified("entrez le nombre d'equations: ")
primal= initialise(x,y)
for j in range(y+1):
    print(primal[j])
dual  = rendre_dual(primal,x,y)
sol_propose = getsol(x)

print("\naffichage du problème primal")
afficher_equation_prblm(primal,x,y,"x")
print("\n******************")
print("**  résolution  **")
print("******************\n")


print("affichage du problème dual")
afficher_equation_prblm(dual,y,x,"y")

print("\n affichage des commentaires : \n")
yi_nulles= afficher_commentaire2(primal,x,y,sol_propose)
temp = commentaire(sol_propose,primal,x,y)
contr_non_sat=temp[0]
check=temp[1]
afficher_commentaire(contr_non_sat)

print("\ncalcul et affichage de l'equation\n")
equation = equation_obtenue(dual,x,y,yi_nulles,contr_non_sat)
afficher_equation(equation ,"y")

print("\nsolution : ")
sol=solution_equation(equation,yi_nulles)
res=afficher_solution(sol,yi_nulles)

print("\ns'assurer que la solution est valide")
check_rest(primal,dual,check, res,sol_propose)
