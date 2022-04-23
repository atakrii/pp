
import glob
import numpy 
import tifffile
import os
import cv2
from PIL import Image 
import time
from moviepy.editor import *
from os import listdir
from os.path import isfile, join

start_time = time.time()

def sortdat(datfiles,b): 
    n =len(datfiles)
    for i in range(n):
        already_sorted = True
        for j in range(n-i-1) :
            datf=datfiles[j]
            datf__ =datfiles[j+1]
            if(float(datf[b:b+2])>float(datf__[b:b+2])):
                datfiles[j], datfiles[j + 1] = datfiles[j + 1], datfiles[j]
                already_sorted= False
        if (already_sorted):
            break
    return datfiles

# Size of the image
width = 2560
height = 2160
#Getting the path by user
user_input = input("Enter the path of your folder: ")
assert os.path.exists(user_input), "Folder not found  at, "+str(user_input)
os.chdir(user_input)
path=os.getcwd() 
subdirs_ = [os.path.join(path, o) for o in os.listdir(path) if 
                               os.path.isdir(os.path.join(path,o))]
print(subdirs_)
input("\n Press Enter to continue...")

for subdir_ in subdirs_:#for each cell
    os.chdir(subdir_) 
    path_ =os.getcwd()
    subdirs= [os.path.join(path_, o) for o in os.listdir(path_) if 
                                   os.path.isdir(os.path.join(path_,o))]
    print('\n')
    print(subdirs)
    for subdir in subdirs:#for each frequency
        print('\n'+ subdir)
        os.chdir(subdir)
        datfiles = glob.glob('*.dat')
        sortdat(datfiles,0)
        
        #Converting dat files to a big single tiff file
        with tifffile.TiffWriter('datfiles.tif', bigtiff=False) as tif:
            for datfile in datfiles:#decrypting the binary file
                data = numpy.fromfile(
                    datfile,
                    count=width * height * 3 // 2,          # 12 bit packed
                    offset=4,                       # 4 byte integer header
                    dtype=numpy.uint8,
                ).astype(numpy.uint16)
                image = numpy.zeros(width * height, numpy.uint16)
                image[0::2] = (data[1::3] & 15) | (data[0::3] << 4)
                image[1::2] = (data[1::3] >> 4) | (data[2::3] << 4)
                image.shape = height, width
                tif.write(
                    image<<4, photometric='minisblack', compression=None, 
                                                                metadata=None)   
        
        #Split the big tiff and then delete it
        img = Image.open('datfiles.tif')
        for i in range(45):
            try:
                img.seek(i)
                img.save('page_%s.tif'%(i,))
            except EOFError:
                break  
        os.remove("datfiles.tif")
        
        #Concatenate the tif files ----> .avi of 3 sec 
        image_folder =os.getcwd()
        video_name = "ConcatenatedVideo.avi"
        
        
        images = [img for img in os.listdir(image_folder) 
                                                  if img.endswith(".tif")]
        images= sortdat(images, 5)
        frame = cv2.imread(os.path.join(image_folder, images[0]))
        height, width, layers = frame.shape
        video = cv2.VideoWriter(video_name, 0, 15, (width,height))
        for image in images:
            video.write(cv2.imread(os.path.join(image_folder, image)))
        cv2.destroyAllWindows()
        video.release()
        
        #delete all the tiff files
        for f in os.listdir(image_folder):          
            if(f.endswith(".tif")):
                os.remove(os.path.join(image_folder, f))            
        print('DONE!')

print('\n ALL FINISHED')

#concatenate final videos

print(path)
os.chdir(path) 
subdirs_1 = [os.path.join(path, o) for o in os.listdir(path) if 
                               os.path.isdir(os.path.join(path,o))]

for subdir_ in subdirs_1:
    os.chdir(subdir_) 
    path_ =os.getcwd()
    subdirs= [os.path.join(path_, o) for o in os.listdir(path_) if 
                                   os.path.isdir(os.path.join(path_,o))]
  
    files=[]
    for subdir in subdirs:
        dir_list = os.listdir(subdir)
        video_path=''
        for f in dir_list:
            if f.endswith('.avi'):
                video_path= os.path.join(subdir, f)
                break
        files.append(video_path)
    clips=[]
    for v in files:
        clips.append(VideoFileClip(v))
    final_video= concatenate_videoclips(clips, method='compose')
    
    final_path =subdir_ +'/output1.avi'
    final_video.write_videofile(final_path,fps=15, threads=1, codec="mjpeg",audio=False)
    print('loop done')

print("--- %s seconds ---" % (time.time() - start_time))        
        
        
        
        
        
        
