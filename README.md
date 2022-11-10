
import requests, json ## importing necessary packages required
import os
%run APIkeys.py ## running our API keys file

from mpl_toolkits.mplot3d import Axes3D ## importing the necessary packages to create figures, dataframes, etc
import matplotlib.pyplot as plt 
import numpy as np 
import pandas as pd 
from sklearn.cluster import KMeans

ID_edamam = os.environ['EDAMAM_API_id'] ## pulling out our edamam API IDs and keys
KEY_edamam = os.environ['EDAMAM_API_key']

def edamam_upc(upc): ## creatunbg a function to list all our items in later
    
    ID_edamam = os.environ['EDAMAM_API_id']
    KEY_edamam = os.environ['EDAMAM_API_key']
    edamamURL = 'https://api.edamam.com/api/food-database/v2/parser?app_id='+ID_edamam+'&app_key='+KEY_edamam+'&upc='
    endURL = '&nutrition-type=cooking&category=packaged-foods'
    totalurl = edamamURL + upc + endURL
    edamam_information = requests.get(totalurl).json()
    if 'error' in edamam_information:
        return edamam_information
    else:
        result = edamam_information['hints'][0]['food']['nutrients']
        
        edamamsize = edamam_information['hints'][0]['food']['servingSizes']
        n = len(edamamsize)
        for i in range(n):
            if edamamsize[i]['label'] == 'Gram':
                result['servingSize_gram'] = edamam_information['hints'][0]['food']['servingSizes'][i]['quantity']
            if edamamsize[i]['label'] == 'Cup':
                result['servingSize_cup'] = edamam_information['hints'][0]['food']['servingSizes'][i]['quantity']
        
        result['label'] = edamam_information['hints'][0]['food']['label']
        result['upc'] = upc
        return result
 
 def multiUPC_edamam(shelf, verbose = False, leftovers = False, source=""):
    
    complete = []   
    missing = [] 
    
    for upc in shelf:
        edamam_info = edamam_upc(upc)
        
        if 'error' in edamam_info:
            missing.append(upc) 
            if verbose:
                print('UPC ',upc, ' does not exist in the API\'s data base')
        
        else:
            if verbose:
                print('UPC ',upc, ' exists in the API\'s data base')
            complete.append(edamam_info)
    
    if len(complete)>0:
        df_complete = pd.DataFrame(complete)
    else:
        df_complete = pd.DataFrame()
    
    df_complete["dataSource"] = pd.Series([source for i in range(len(df_complete.index))]) ## creating a necessary function which we will use later
    
    if leftovers:
        return df_complete,missing
    else:
        return df_complete
 
 Kroger_UPCSs = ["0003800019904", "0001600012495", "0001600012254", "0001600012399", "0003800020074", "0088491212971", "0088491235916", "0088491235916","0001600012222", "0003800019886", "0088491211171", "0001111086705", "0001600010371", "0088491212951", "0003800019934", "0088491235915", "0003800019888", "0001111003245", "0001600012125", "0001600016361", "0003800026996", "0003000006322", "0003000057331", "0001600012224", "0003800019849", "0088491200672", "0088491227311", "0001111089823", "0088491211762", "0003000057323", "0003800020045", "0088397814726", "0003800019851", "0007061700623", "0003000056238", "0001862770321", "1002190845556", "0001862770357", "0088397812911", "0005844977020", "0005844986004", "0088491235916", "0001600012222", "0003800019886", "0088491211171", "0001111086705", "0001600010371", "0088491212951", "0003800019934", "0088491235915", "0003800019888", "0001111003245", "0001600012125", "0001600016361", "0003800026996", "0003000006322", "0003000057331", "0001600012224", "0003800019849", "0088491200672", "0088491227311", "0001111089823", "0088491211762", "0003000057323", "0003800020045", "0088397814726", "0003800019851", "0007061700623", "0003000056238", "0001862770321", "1002190845556", "0001862770357", "0088397812911", "0005844977020", "0005844986004", " 0001111010005", " 0003800019906", "0003800020003", "0003800019987", "099482438937"] ## picking out UPC codes of cereals from Kroger
 
 Kroger_cereal, Kroger_leftovers = multiUPC_edamam(Kroger_UPCSs, verbose = True, leftovers = True, source = "Kroger") ## checking whether these codes exist in the edamam API database
 
 Kroger_cereal ## printing out our data frame of Kroger cereals
 
 Amazon_UPCS = ["099482438937", "099482438869", "099482448639", "099482439057", "099482448646", "099482438982", "099482439040", "018627703570", "860002152424"] ## printing out UPC codes of cereal from Amazon
 
 Amazon_cereal, Amazon_leftovers = multiUPC_edamam(Amazon_UPCS, verbose = True, leftovers = True, source = "Amazon") ## checking whether these codes exist in the edamam API database
 
 Amazon_cereal ## printing out the Amazon cereal dataframe
 
 ## 1.b
Cereal_combined = Kroger_cereal.append(Amazon_cereal, ignore_index = True) ## creating a combined dataframe with information from both our Kroger and Amazon cereals

Cereal_combined = Cereal_combined[['ENERC_KCAL', 'FAT', 'FASAT', 'CHOCDF', 'FIBTG', 'SUGAR', 'dataSource']] ## printing out only select columns from our combined data frame

Cereal_combined ## printing out our new dataframe

Cereal_combined = Cereal_combined.dropna() ## dropping our missing values from our combined dataframe

# 1.c
Cereal_combined1 = Cereal_combined.describe() ## creating a table of summary descriptive statistics

Cereal_combined1 ## summarizing our data descriptive statistics

Cereal_combined.isnull().sum() ## confirming that the number of missing oservations is equal to 38-count

## 2.1 
facts = ['ENERC_KCAL', 'FAT', 'FASAT', 'CHOCDF', 'FIBTG', 'SUGAR'] ## sub-selecting only certain variables

Cereal_combined[facts].corr() ## printing out correlations of our selected variables 

facts = ['ENERC_KCAL', 'FAT', 'FASAT', 'CHOCDF', 'FIBTG', 'SUGAR'] ## printing out a plot of carbs and energy, two variables with an extremely high correlation
Cereal_small = Cereal_combined[facts]
plt.scatter(data=Cereal_combined, x=facts[3], y=facts[0])
plt.title("Relationship between Carbs and Energy Between Cereal Sold Through Kroger and Amazon")
plt.xlabel(facts[3])
plt.ylabel(facts[0])
plt.show()

plt.scatter(data=Cereal_combined, x=facts[5], y=facts[1]) ## creating a scatterplot of sugar and fat, two variables with an extremely low correlation
plt.title("Relationship between Sugar and Fat Between Cereal Sold Through Kroger and Amazon")
plt.xlabel(facts[5])
plt.ylabel(facts[1])
plt.show()

plt.scatter(data=Cereal_combined, x=facts[5], y=facts[4]) ## creating a scatterplot of sugar and fat, two variables with an extremely low correlation
plt.title("Relationship between Sugar and Fiber Between Cereal Sold Through Kroger and Amazon")
plt.xlabel(facts[5])
plt.ylabel(facts[4])
plt.show()

# 2.2
m = Cereal_small.min()
M = Cereal_small.max()

def  rescaleFeatures(Cereal_combined_in):

    Cereal_combined_out = Cereal_combined_in 
    m = Cereal_combined_out.min() 
    M = Cereal_combined_out.max() 
    for i in range(len(Cereal_combined_out.columns)):
        Cereal_combined_out.iloc[:,i] = (Cereal_combined_out.iloc[:,i] - m[i])/(M[i]-m[i])
    
    return Cereal_combined_out,m,M
    
def upscaleCentroids(centroids_in,m,M):
    
    centroid_out = centroids_in
    n_centroids = len(centroids_in) 
    n_features = len(centroids_in[0])
    for i in range(n_centroids):
        for j in range(n_features):
            centroid_out[i][j] = (centroid_out[i][j] )*(M[j]-m[j]) + m[j] 
    
    return centroid_out
    
 df1,m,M = rescaleFeatures(Cereal_small)
 
 k = 5
kmeans = KMeans(n_clusters=k, init='k-means++', random_state=0).fit(df1)

centroids1 = upscaleCentroids(kmeans.cluster_centers_,m,M)

plt.scatter(data=Cereal_combined, x=facts[5], y=facts[4],c=kmeans.labels_)
plt.scatter(kmeans.cluster_centers_[:,0], kmeans.cluster_centers_[:,1],marker="X", c="r", s=80, label="centroids")
plt.title(f"Position of {n} cereals in {k} clusters")
plt.xlabel(facts[5])
plt.ylabel(facts[4])
plt.show()

print(f"The inertia measure for k={k} is {kmeans.inertia_}")

inertia = []
n_clusters = []
for i in range(1,13):
    kmeans = KMeans(n_clusters=i, init='k-means++', random_state=0).fit(df1)
    inertia.append(kmeans.inertia_)
    n_clusters.append(i)
    
    
plt.plot(n_clusters,inertia, marker='o')
plt.xlabel('Number of clusters')
plt.ylabel('Distortion')
plt.title("Inertia of K-means clustering")
plt.show()
