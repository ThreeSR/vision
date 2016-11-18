# torch-vision

This repository consists of:

- [vision.datasets](#datasets) : Data loaders for popular vision datasets
- [vision.transforms](#transforms) : Common image transformations such as random crop, rotations etc.
- [vision.utils](#utils) : Useful stuff such as saving tensor (3 x H x W) as image to disk, given a mini-batch creating a grid of images, etc.
- `[WIP] vision.models` : Model definitions and Pre-trained models for popular models such as AlexNet, VGG, ResNet etc.

# Installation

Binaries:

```bash
conda install torchvision -c https://conda.anaconda.org/t/6N-MsQ4WZ7jo/soumith
```

From Source:

```bash
pip install -r requirements.txt
pip install .
```

# Datasets

The following dataset loaders are available:

- [COCO (Captioning and Detection)](#coco)
- [LSUN Classification](#lsun)
- [ImageFolder](#imagefolder)
- [Imagenet-12](#imagenet-12)
- [CIFAR10 and CIFAR100](#cifar)

Datasets have the API:
- `__getitem__`
- `__len__`
They all subclass from `torch.utils.data.Dataset`
Hence, they can all be multi-threaded (python multiprocessing) using standard torch.utils.data.DataLoader.

For example:

`torch.utils.data.DataLoader(coco_cap, batch_size=args.batchSize, shuffle=True, num_workers=args.nThreads)`

In the constructor, each dataset has a slightly different API as needed, but they all take the keyword args:

- `transform` - a function that takes in an image and returns a transformed version
  - common stuff like `ToTensor`, `RandomCrop`, etc. These can be composed together with `transforms.Compose` (see transforms section below)
- `target_transform` - a function that takes in the target and transforms it. For example, take in the caption string and return a tensor of word indices.

### COCO

This requires the [COCO API to be installed](https://github.com/pdollar/coco/tree/master/PythonAPI)

#### Captions:

`dset.CocoCaptions(root="dir where images are", annFile="json annotation file", [transform, target_transform])`

Example:

```python
import torchvision.datasets as dset
import torchvision.transforms as transforms
cap = dset.CocoCaptions(root = 'dir where images are',
                        annFile = 'json annotation file',
                        transform=transforms.ToTensor())

print('Number of samples: ', len(cap))
img, target = cap[3] # load 4th sample

print("Image Size: ", img.size())
print(target)
```

Output:

```
Number of samples: 82783
Image Size: (3L, 427L, 640L)
[u'A plane emitting smoke stream flying over a mountain.',
u'A plane darts across a bright blue sky behind a mountain covered in snow',
u'A plane leaves a contrail above the snowy mountain top.',
u'A mountain that has a plane flying overheard in the distance.',
u'A mountain view with a plume of smoke in the background']
```

#### Detection:
`dset.CocoDetection(root="dir where images are", annFile="json annotation file", [transform, target_transform])`

### LSUN

`dset.LSUN(db_path, classes='train', [transform, target_transform])`

- db_path = root directory for the database files
- classes =
  - 'train' - all categories, training set
  - 'val' - all categories, validation set
  - 'test' - all categories, test set
  - ['bedroom_train', 'church_train', ...] : a list of categories to load


### CIFAR

`dset.CIFAR10(root, train=True, transform=None, target_transform=None, download=False)`

`dset.CIFAR100(root, train=True, transform=None, target_transform=None, download=False)`

- `root` : root directory of dataset where there is folder `cifar-10-batches-py`
- `train` : `True` = Training set, `False` = Test set
- `download` : `True` = downloads the dataset from the internet and puts it in root directory. If dataset already downloaded, does not do anything.

### ImageFolder

A generic data loader where the images are arranged in this way:

```
root/dog/xxx.png
root/dog/xxy.png
root/dog/xxz.png

root/cat/123.png
root/cat/nsdf3.png
root/cat/asd932_.png
```

`dset.ImageFolder(root="root folder path", [transform, target_transform])`

It has the members:

- `self.classes` - The class names as a list
- `self.class_to_idx` - Corresponding class indices
- `self.imgs` - The list of (image path, class-index) tuples


### Imagenet-12

This is simply implemented with an ImageFolder dataset.

The data is preprocessed [as described here](https://github.com/facebook/fb.resnet.torch/blob/master/INSTALL.md#download-the-imagenet-dataset)

[Here is an example](https://github.com/pytorch/examples/blob/27e2a46c1d1505324032b1d94fc6ce24d5b67e97/imagenet/main.py#L48-L62).


# Transforms

Transforms are common image transforms.
They can be chained together using `transforms.Compose`

### `transforms.Compose`

One can compose several transforms together.
For example.

```python
transform = transforms.Compose([
    transforms.RandomSizedCrop(224),
    transforms.RandomHorizontalFlip(),
    transforms.ToTensor(),
    transforms.Normalize(mean = [ 0.485, 0.456, 0.406 ],
                          std = [ 0.229, 0.224, 0.225 ]),
])
```

## Transforms on PIL.Image

### `Scale(size, interpolation=Image.BILINEAR)`
Rescales the input PIL.Image to the given 'size'.
'size' will be the size of the smaller edge.

For example, if height > width, then image will be
rescaled to (size * height / width, size)
- size: size of the smaller edge
- interpolation: Default: PIL.Image.BILINEAR

### `CenterCrop(size)` - center-crops the image to the given size
Crops the given PIL.Image at the center to have a region of
the given size. size can be a tuple (target_height, target_width)
or an integer, in which case the target will be of a square shape (size, size)

### `RandomCrop(size)`
Crops the given PIL.Image at a random location to have a region of
the given size. size can be a tuple (target_height, target_width)
or an integer, in which case the target will be of a square shape (size, size)

### `RandomHorizontalFlip()`
Randomly horizontally flips the given PIL.Image with a probability of 0.5

### `RandomSizedCrop(size, interpolation=Image.BILINEAR)`
Random crop the given PIL.Image to a random size of (0.08 to 1.0) of the original size
and and a random aspect ratio of 3/4 to 4/3 of the original aspect ratio

This is popularly used to train the Inception networks
- size: size of the smaller edge
- interpolation: Default: PIL.Image.BILINEAR

## Transforms on torch.*Tensor

### `Normalize(mean, std)`
Given mean: (R, G, B) and std: (R, G, B), will normalize each channel of the torch.*Tensor, i.e. channel = (channel - mean) / std

## Conversion Transforms
- `ToTensor()` - Converts a PIL.Image (RGB) or numpy.ndarray (H x W x C) in the range [0, 255] to a torch.FloatTensor of shape (C x H x W) in the range [0.0, 1.0]
- `ToPILImage()` - Converts a torch.*Tensor of range [0, 1] and shape C x H x W or numpy ndarray of dtype=uint8, range[0, 255] and shape H x W x C to a PIL.Image of range [0, 255]


# Utils

### make_grid(tensor, nrow=8, padding=2)
Given a 4D mini-batch Tensor of shape (B x C x H x W), makes a grid of images

### save_image(tensor, filename, nrow=8, padding=2)
Saves a given Tensor into an image file.

If given a mini-batch tensor, will save the tensor as a grid of images.
