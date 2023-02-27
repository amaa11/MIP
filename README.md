# MIP

```
X_train = path
X_test = path
y_train = path
y_test = path
Columns_list = list(X_train.columns.values)
#########################################
dicts = dict()
model_1= LogisticRegression(C=0.1)                                ## defin a model
model_1.fit(X_train, y_train.values.ravel())                      ## fit the data to the model
ypred_test = model_1.predict (X_test)                             ## predict test data
    
#prepare the data for shap
masker = shap.maskers.Independent(data = X_train)
    
# fit to shap
explainer = shap.LinearExplainer(model_1, masker = masker)
    
# apply shap to test data
shap_values = explainer.shap_values(X_test)
    
# the follwoing steps to represents the SHAP outcome (informative predictors) as datafram
vals= np.abs(shap_values).mean(0)
SHAP_out = pd.DataFrame(list(zip(X_train.columns,vals)),columns=['SHAP','feature_importance_vals'])
SHAP_out.sort_values(by=['feature_importance_vals'],ascending=False,inplace=True)
SHAP_out.reset_index(inplace=True, drop=True)
SHAP_out.index = np.arange(1, len(SHAP_out) + 1)


    
while X_train.shape[1] != 1 :
    print('Iteration: ', X_train.shape[1])                          ## number of features in each iteration
    model= LogisticRegression(C=0.1)                                ## defin a model
    model.fit(X_train, y_train.values.ravel())                      ## fit the data to the model
    ypred_test = model.predict (X_test)                             ## predict test data
    
    #prepare the data for shap
    masker = shap.maskers.Independent(data = X_train)
    
    # fit shap
    explainer = shap.LinearExplainer(model, masker = masker)
    
    # apply shap to test data
    shap_values = explainer.shap_values(X_test)
    
    # the follwoing steps to represents the SHAP outcome (informative predictors) as datafram
    vals= np.abs(shap_values).mean(0)
    feature_importance = pd.DataFrame(list(zip(X_train.columns,vals)),columns=['Features','feature_importance_vals'])
    feature_importance.sort_values(by=['feature_importance_vals'],ascending=False,inplace=True)
    feature_importance.reset_index(inplace=True, drop=True)
    feature_importance.index = np.arange(1, len(feature_importance) + 1)
    #print(feature_importance)
    FEATURES = feature_importance[['Features']]
    #print(FEATURES)
    print('=====================')
    for i in range(FEATURES.shape[0]):
        #print(FEATURES.iloc[i]['Features'], end = " ")
        #print(FEATURES.iloc[i].name, end = " ")
        Frac = FEATURES.iloc[i].name/X_train.shape[1]
        #print(Frac)
        #for key in dicts:
        if FEATURES.iloc[i]['Features'] in dicts:
            dicts[FEATURES.iloc[i]['Features']].append(Frac)
        else:
            dicts[FEATURES.iloc[i]['Features']] = [Frac]
        
        #dicts[FEATURES.iloc[i]['Features']] = FEATURES.iloc[i].name/X_train.shape[1]
    
    
    # identify the most informative predictor
    Top_feature = feature_importance.iloc[0]['Features']
    
    # remove the most informative predictor from train and test data
    X_train.drop([Top_feature], axis=1, inplace=True)
    X_train.reset_index(inplace=True, drop=True)
    X_test.drop([Top_feature], axis=1, inplace=True)
    X_test.reset_index(inplace=True, drop=True)
out = {k: [sum(dicts[k])] for k in dicts.keys()}
out = pd.DataFrame(out.items(), columns=['Modified SHAP', 'Score'])
out['Score'] = out['Score'].str[0]
out.sort_values(by=['Score'], inplace=True)
out.reset_index(inplace=True, drop=True)
out.index = np.arange(1, len(out) + 1)
out = pd.concat([out, SHAP_out[['SHAP']]], axis=1)
```
And then print the new order along side with the initial order and the standard deviation

```
print(out)
print(out['Score'].std())
```
