# PyCuda

One of the reasons why I wanted to get into Python is because I wanted to get back to Cuda development. I've done Cuda development 5 years ago
when I used VS 2013 and 2015, when the Cuda development kit made it easy to create an app in VS that could run some code on the GPU. Now, it seems
like the latest Cuda development kit's VS integration won't work in VS 2019.

So, I came across PyCuda where I would write Python code to actually communicate with the GPU, and it looks VERY EASY compared to what I had to do
in VS with C and C++. But figuring out how to install the whole system was a pain, but I finally figured it out!

I first installed Python right off the Python download page. Then, I followed instructions [from here](https://www.ibm.com/developerworks/community/blogs/jfp/entry/Installing_PyCUDA_On_Anaconda_For_Windows?lang=en).
The article is quite old, at 4 years at the time of writing this article, but it still worked. The websites it referenced are still there.

I already had VS 2019 installed in my system, and I downloaded the Cuda development toolkit form NVIDIA, version 10.2.

Regarding step 2, though, since I had VS 2019, I needed to add this location to my system's `Path` environment variables:

`C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.24.28314\bin\Hostx64\x64`.

This contained the executable `cl.exe`, which PyCuda and `nvcc` would need. I don't know exactly this file's purpose is, but it is required, or else
PyCuda won't work.

Then came the Anaconda installation, which seemed overkill. I chose the 64-bit distribution with Python 3.7. Though I already had Python 3.8.1 already
installed, I still went ahead with the Anaconda installation. I believe that this package installed `numpy`, a library necessary for manipulating
numbers and arrays that `pycuda` requires.

Step 4 was to grab the pre-built binary for `pycuda`. This is where things got tricky and it took me quite a while to figure out. I chose to download
the latest `.whl` - `pycuda‑2019.1.2+cuda102‑cp38‑cp38‑win_amd64.whl`. I have Cuda 10.2 installed, I had Python 3.8.1 installed, and I have a 64-bit
system, so this had to be the right file. So, I installed downloaded it and installed it using `pip`, and I got an error saying that `....whl is not
supported wheel on this platform`. So at this time, I can't set up `pycuda`. What happened?

On (python.org/downloads)[https://www.python.org/downloads], there's a yellow button with the text `Download Python 3.8.1` on it (at the time of this
writing). Clicking this button would automatically save `python-3.8.1.exe` on our machines. But little do we know that this is 32-bit Python, since
nowhere does it say that. That's what I downloaded and installed, and I was unaware that I was running 32-bit Python, which that `.whl` file above
was NOT compatible with - that's why it was giving me that `is not supported wheel on this platform` error. So, instead, I scrolled down a little
down to the section entitled `Looking for a specific release?`. I clicked on the first item - `Python 3.8.1`, which took me to a different page.
On the very bottom of that page is a file listing. I chose the option `Windows x86-64 executable installer` which IS the correct one. So, I
uninstalled the 32-bit Python, and installed the 64-bit one. I went back to install the `.whl` file with `pip` at the command prompt, and finally,
it installed!

So, now, I copied some `pycuda` code online, put it in my project, and ran it. It works as expected! Now, I can go ahead and focus on `pycuda`!



