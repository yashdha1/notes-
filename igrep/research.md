# Grep Functionality
1. grep -i -> ignore case 
does not matter to it... 
3. grep -c -> prints the freauency

4. link of the image shall be given with the search result. 


# TODO : 

1. first of beutify the current results.
2. add multiprocessing and graphccs for better and lot of images.
3. add argparse for better input and interaction with application.
4. also add the project to the path for best results.
5. added fucntionality to recursively find the .png and .jpg file to make its context. 
        a. jaybe-jaybe not : 
        b. actually can prolly limit it to two folders only : 
* commands : 

A. normal search : 
    1. igrep "pattern_search"           : normal 
    2. igrep -i "pattern_search"        : ignore casing
    3. igrep -c "pattern"               : count the occurences
B. semantics search :
    3. igrep -s "text"                  : semantic search default top_5 
    4. igrep -s "text" <topk>           : semantic search default top_k
C. sync the images
    5. igrep sync                       : sync the images present in the folder to have in the db
D.  model installation and db setup  (hide all the ugly shit)
    6. igrep setup                      


More features to be must added: \
