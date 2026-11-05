Pour construire l'algorithme, nous avons suivi les étapes suivantes :

1-scission des 52 batchs ou coupures en 44 training et 8 holdouts
2-entrainement log-regression avec cross-validation en 5 plis (k-fold = 5)
3-évaluation Holdout : PR-AUC 0.1346 pour une prévalence de 0.0335 (lift 4.01 ; un modèle au hasard a un lift de 1)



A expliquer : 
construction algo, sortie : 

Validation croisée temporelle (PR-AUC moyenne par réglage) :
     preparation       C  pr_auc_cv  ecart_type  lift_cv  plis
        standard  0.1000     0.0681      0.0412   2.3907     4
        standard  1.0000     0.0681      0.0411   2.3904     4
        standard 10.0000     0.0681      0.0411   2.3906     4
        standard  0.0100     0.0679      0.0409   2.3846     4
        standard  0.0010     0.0669      0.0393   2.3705     4
arcsinh_standard  0.0100     0.0634      0.0400   2.3557     4
arcsinh_standard  0.0010     0.0633      0.0404   2.3398     4
arcsinh_standard  1.0000     0.0633      0.0399   2.3519     4
arcsinh_standard  0.1000     0.0633      0.0399   2.3516     4
arcsinh_standard 10.0000     0.0633      0.0399   2.3514     4

evaluation holdout : sortie :
Table des seuils (holdout, 8 semaines) :
 seuil  precision  rappel  alertes_par_semaine
 0.051      0.034   1.000             1167.500
 0.100      0.037   0.978             1048.625
 0.150      0.038   0.975              999.875
 0.200      0.040   0.952              934.625
 0.250      0.041   0.911              861.875
 0.300      0.046   0.879              754.625
 0.350      0.051   0.815              626.750
 0.400      0.056   0.796              555.250
 0.450      0.061   0.768              490.625
 0.500      0.068   0.739              428.125
 0.550      0.076   0.713              368.875
 0.600      0.089   0.659              290.000
 0.650      0.119   0.510              168.250
 0.700      0.161   0.242               59.000
 0.751      0.207   0.080               15.125
 0.801      0.356   0.051                5.625
 0.852      0.450   0.029                2.500
 0.903      0.500   0.013                1.000
 0.950      0.667   0.006                0.375
→ seuil proposé (précision max sous un rappel ≥ 20%) : 0.708 (précision 0.173, rappel 0.217, 49.1 alertes par semaine)
