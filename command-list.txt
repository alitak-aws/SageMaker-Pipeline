'''Selecting columns
'''
df = df[['containerid', 'eventtimeutc', 'eventlatitude', 'eventlongitude', 'speed']].copy()



'''Converting timestamp to DateTime 
'''
import pandas
df['eventtimeutc']= pandas.to_datetime(df.eventtimeutc, errors='raise', utc=True)


'''Limiting timestamp to before ‘2023’
'''
df = df[df.eventtimeutc<'2023'].copy()


''' Converting lat-long to distance on sphere
'''
import pandas as pd
import numpy as np

def haversine(lat1, lon1, lat2, lon2, to_radians=True, earth_radius=6371):
    """
    slightly modified version: of http://stackoverflow.com/a/29546836/2901002
    Calculate the great circle distance between two points
    on the earth (specified in decimal degrees or in radians)
    All (lat, lon) coordinates must have numeric dtypes and be of equal length.
    """
    if to_radians:
        lat1, lon1, lat2, lon2 = np.radians([lat1, lon1, lat2, lon2])

    a = np.sin((lat2-lat1)/2.0)**2 + np.cos(lat1) * np.cos(lat2) * np.sin((lon2-lon1)/2.0)**2

    return earth_radius * 2 * np.arcsin(np.sqrt(a))

df['dist'] = -100
g1 = df.groupby('containerid')

for i in g1.groups.keys():
    tmp = df[df.containerid==i]
    ind = g1.get_group(i).index.tolist()
    df.loc[ind, 'dist'] = haversine(tmp['eventlatitude'], tmp['eventlongitude'],
                       tmp['eventlatitude'].shift(), tmp['eventlongitude'].shift(),
                       to_radians=False)




''' Synchronizing data 
'''
import pandas as pd

df3 = pd.DataFrame()
gr = df.groupby(‘containerid’)

for i in gr.groups.keys():
    print(i)
    ind = gr.get_group(i).index.tolist()
    df2 = df.loc[ind].set_index('eventtimeutc').resample('D').mean().reset_index() ## resample('15min’) for 15 min sampling
    df2['containerid'] = i
    df3 = pd.concat([df3,df2])

df = df3.copy()


''' Conditional labeling
'''
import numpy as np
df['isDwelling'] = np.where(df['dist']>10, 1, 0)



