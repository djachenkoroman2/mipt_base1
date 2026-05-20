# Перечень вопросов, заданий, тем для подготовки к текущему контролю

## Осенний семестр

## Задание 1. Обрезка облака точек по ограничивающему боксу

### Исходная формулировка

> Разработать программу на языке Python для «Обрезки» облака точек по ограничивающему боксу.

### Краткая цель и идея решения

Цель задания — выделить из исходного облака только те точки, которые попадают внутрь заданного ограничивающего бокса. В учебном примере используется осесимметричный ограничивающий бокс `Axis-Aligned Bounding Box, AABB`, заданный минимальной и максимальной координатами по осям `X`, `Y`, `Z`.

Идея решения простая:

1. Сформировать или загрузить облако точек.
2. Задать границы бокса.
3. Отфильтровать точки по условию попадания внутрь бокса.
4. Сохранить результат и визуально сравнить исходное и обрезанное облака.

### Какие библиотеки используются и зачем

- `numpy` — работа с массивами координат и логическими масками.
- `open3d` — представление облака точек и сохранение результата в формат `PLY`.
- `matplotlib` — простая 3D-визуализация, которая нормально работает в Google Colab.

### Готовый код для Google Colab

Ячейка установки зависимостей:

```bash
!pip -q install open3d
```

Основной код:

```python
import numpy as np
import open3d as o3d
import matplotlib.pyplot as plt

rng = np.random.default_rng(42)


def make_scene(n_ground=6000, n_box=5000, n_sphere=4000):
    ground_x = rng.uniform(-2.5, 2.5, n_ground)
    ground_y = rng.uniform(-2.5, 2.5, n_ground)
    ground_z = rng.normal(0.0, 0.005, n_ground)
    ground = np.column_stack([ground_x, ground_y, ground_z])

    box = rng.uniform([-0.8, -0.6, 0.2], [0.4, 0.6, 1.1], size=(n_box, 3))

    phi = rng.uniform(0, 2 * np.pi, n_sphere)
    cos_theta = rng.uniform(-1, 1, n_sphere)
    u = rng.uniform(0, 1, n_sphere)
    theta = np.arccos(cos_theta)
    r = 0.45 * np.cbrt(u)
    sphere = np.column_stack([
        1.2 + r * np.sin(theta) * np.cos(phi),
        -0.6 + r * np.sin(theta) * np.sin(phi),
        0.7 + r * np.cos(theta),
    ])

    points = np.vstack([ground, box, sphere])
    labels = np.concatenate([
        np.zeros(len(ground), dtype=np.int64),
        np.ones(len(box), dtype=np.int64),
        np.full(len(sphere), 2, dtype=np.int64),
    ])
    return points, labels


def draw_bbox(ax, min_bound, max_bound):
    x0, y0, z0 = min_bound
    x1, y1, z1 = max_bound
    corners = np.array([
        [x0, y0, z0], [x1, y0, z0], [x1, y1, z0], [x0, y1, z0],
        [x0, y0, z1], [x1, y0, z1], [x1, y1, z1], [x0, y1, z1],
    ])
    edges = [
        (0, 1), (1, 2), (2, 3), (3, 0),
        (4, 5), (5, 6), (6, 7), (7, 4),
        (0, 4), (1, 5), (2, 6), (3, 7),
    ]
    for i, j in edges:
        ax.plot(*zip(corners[i], corners[j]), color="black", linewidth=1.5)


def plot_cloud(ax, points, labels, title, min_bound=None, max_bound=None, max_points=7000):
    if len(points) > max_points:
        idx = rng.choice(len(points), size=max_points, replace=False)
        points = points[idx]
        labels = labels[idx]

    palette = np.array(["#9e9e9e", "#1f77b4", "#ff7f0e"])
    ax.scatter(points[:, 0], points[:, 1], points[:, 2], c=palette[labels], s=2)
    if min_bound is not None and max_bound is not None:
        draw_bbox(ax, min_bound, max_bound)
    ax.set_title(title)
    ax.set_xlabel("X")
    ax.set_ylabel("Y")
    ax.set_zlabel("Z")
    ax.set_box_aspect((1, 1, 0.6))


points, labels = make_scene()

min_bound = np.array([-1.0, -0.9, 0.0])
max_bound = np.array([0.8, 0.9, 1.0])

pcd = o3d.geometry.PointCloud()
pcd.points = o3d.utility.Vector3dVector(points)

bbox = o3d.geometry.AxisAlignedBoundingBox(min_bound=min_bound, max_bound=max_bound)
cropped_pcd = pcd.crop(bbox)

mask = np.all((points >= min_bound) & (points <= max_bound), axis=1)
cropped_points = points[mask]
cropped_labels = labels[mask]

print(f"Исходное число точек: {len(points)}")
print(f"После обрезки логической маской: {len(cropped_points)}")
print(f"Проверка через Open3D: {len(cropped_pcd.points)}")

fig = plt.figure(figsize=(14, 6))
ax1 = fig.add_subplot(121, projection="3d")
ax2 = fig.add_subplot(122, projection="3d")
plot_cloud(ax1, points, labels, "Исходное облако и ограничивающий бокс", min_bound, max_bound)
plot_cloud(ax2, cropped_points, cropped_labels, "Результат обрезки")
plt.tight_layout()
plt.show()

o3d.io.write_point_cloud("cropped_cloud.ply", cropped_pcd)
print("Результат сохранён в файл cropped_cloud.ply")
```

### Как запустить код и какой результат должен получиться

1. Открыть Google Colab.
2. Выполнить ячейку установки `open3d`.
3. Выполнить основной код.

В результате:

- будет создано учебное облако точек;
- на первом графике отобразится исходное облако и контур ограничивающего бокса;
- на втором графике отобразятся только точки внутри бокса;
- в рабочем каталоге появится файл `cropped_cloud.ply`.

### Краткие замечания об ограничениях, допущениях или возможных улучшениях

- В примере используется синтетическое облако, потому что исходный датасет в задании не задан.
- Показан осесимметричный бокс `AABB`. Для произвольного повёрнутого бокса можно использовать `OrientedBoundingBox`.
- Для реальных данных вместо `make_scene()` можно читать файл через `o3d.io.read_point_cloud("input.ply")`.

## Задание 2. Операции разреживания облака точек

### Исходная формулировка

> Разработать программу на языке Python, реализующую операции разреживания облака точек.

### Краткая цель и идея решения

Цель задания — уменьшить количество точек в облаке так, чтобы сохранить его форму настолько, насколько это возможно. В учебном материале показаны три типовые операции разреживания:

1. случайная выборка `Random Sampling`;
2. воксельное разреживание `Voxel Grid Downsampling`;
3. выборка самых удалённых точек `Farthest Point Sampling, FPS`.

Это хороший учебный набор, потому что методы отличаются по качеству результата и вычислительной стоимости.

### Какие библиотеки используются и зачем

- `numpy` — реализация случайной выборки и `FPS`.
- `open3d` — воксельное разреживание через готовый геометрический инструмент.
- `matplotlib` — визуальное сравнение исходного и разреженных облаков.
- `time` — измерение времени выполнения операций.

### Готовый код для Google Colab

Ячейка установки зависимостей:

```bash
!pip -q install open3d
```

Основной код:

```python
import time
import numpy as np
import open3d as o3d
import matplotlib.pyplot as plt

rng = np.random.default_rng(7)


def make_scene(n_ground=5000, n_box=2500, n_sphere=2000, n_cylinder=2500):
    ground_x = rng.uniform(-2.5, 2.5, n_ground)
    ground_y = rng.uniform(-2.5, 2.5, n_ground)
    ground_z = rng.normal(0.0, 0.004, n_ground)
    ground = np.column_stack([ground_x, ground_y, ground_z])

    box = rng.uniform([-1.0, -0.8, 0.1], [0.2, 0.8, 1.1], size=(n_box, 3))

    phi = rng.uniform(0, 2 * np.pi, n_sphere)
    cos_theta = rng.uniform(-1, 1, n_sphere)
    theta = np.arccos(cos_theta)
    sphere = np.column_stack([
        1.4 + 0.45 * np.sin(theta) * np.cos(phi),
        -0.5 + 0.45 * np.sin(theta) * np.sin(phi),
        0.8 + 0.45 * np.cos(theta),
    ])

    angles = rng.uniform(0, 2 * np.pi, n_cylinder)
    z = rng.uniform(0.0, 1.3, n_cylinder)
    cylinder = np.column_stack([
        -1.5 + 0.35 * np.cos(angles),
        1.0 + 0.35 * np.sin(angles),
        z,
    ])

    points = np.vstack([ground, box, sphere, cylinder])
    return points


def farthest_point_sampling(points, n_samples, seed=0):
    rng_local = np.random.default_rng(seed)
    n_points = len(points)
    selected = np.empty(n_samples, dtype=np.int64)
    distances = np.full(n_points, np.inf)
    selected[0] = rng_local.integers(0, n_points)
    farthest = points[selected[0]]

    for i in range(1, n_samples):
        dist = np.sum((points - farthest) ** 2, axis=1)
        distances = np.minimum(distances, dist)
        selected[i] = np.argmax(distances)
        farthest = points[selected[i]]

    return selected


def plot_cloud(ax, points, title, max_points=5000):
    if len(points) > max_points:
        idx = rng.choice(len(points), size=max_points, replace=False)
        points = points[idx]
    ax.scatter(points[:, 0], points[:, 1], points[:, 2], c=points[:, 2], cmap="viridis", s=2)
    ax.set_title(f"{title}\n{len(points)} точек на графике")
    ax.set_xlabel("X")
    ax.set_ylabel("Y")
    ax.set_zlabel("Z")
    ax.set_box_aspect((1, 1, 0.6))


points = make_scene()
pcd = o3d.geometry.PointCloud()
pcd.points = o3d.utility.Vector3dVector(points)

target_points = 2000

t0 = time.perf_counter()
random_idx = rng.choice(len(points), size=target_points, replace=False)
random_points = points[random_idx]
t_random = time.perf_counter() - t0

t0 = time.perf_counter()
voxel_points = np.asarray(pcd.voxel_down_sample(voxel_size=0.08).points)
t_voxel = time.perf_counter() - t0

t0 = time.perf_counter()
fps_idx = farthest_point_sampling(points, n_samples=target_points, seed=7)
fps_points = points[fps_idx]
t_fps = time.perf_counter() - t0

print(f"Исходное облако: {len(points)} точек")
print(f"Random Sampling: {len(random_points)} точек, время {t_random:.3f} c")
print(f"Voxel Grid: {len(voxel_points)} точек, время {t_voxel:.3f} c")
print(f"Farthest Point Sampling: {len(fps_points)} точек, время {t_fps:.3f} c")

fig = plt.figure(figsize=(14, 10))
axes = [
    fig.add_subplot(221, projection="3d"),
    fig.add_subplot(222, projection="3d"),
    fig.add_subplot(223, projection="3d"),
    fig.add_subplot(224, projection="3d"),
]

plot_cloud(axes[0], points, "Исходное облако")
plot_cloud(axes[1], random_points, "Случайная выборка")
plot_cloud(axes[2], voxel_points, "Воксельное разреживание")
plot_cloud(axes[3], fps_points, "FPS")

plt.tight_layout()
plt.show()
```

### Как запустить код и какой результат должен получиться

1. Выполнить ячейку установки `open3d`.
2. Выполнить основной код.

Результат:

- в консоль будут выведены размеры облака после каждого метода разреживания и примерное время работы;
- на рисунке будут видны отличия между подходами:
  - случайная выборка сохраняет форму, но распределяет точки неравномерно;
  - `Voxel Grid` хорошо выравнивает плотность;
  - `FPS` обычно лучше сохраняет геометрию при заданном числе точек.

### Краткие замечания об ограничениях, допущениях или возможных улучшениях

- Реализация `FPS` сделана в учебном виде и имеет высокую вычислительную стоимость на очень больших облаках.
- Для промышленных облаков часто применяют сочетание методов: сначала `Voxel Grid`, потом `FPS`.
- Для облаков с атрибутами `RGB`, нормалями и интенсивностью стоит разреживать данные так, чтобы атрибуты не терялись или корректно агрегировались.

## Задание 3. Класс набора данных для сегментации и классификации облаков точек

### Исходная формулировка

> Реализовать класс для набора данных для решения задач сегментации и классификации облаков точек.

### Краткая цель и идея решения

Цель задания — построить универсальный класс датасета, который может использоваться как в задаче классификации облаков, так и в задаче посегментной разметки точек. В учебном варианте:

1. автоматически создаётся небольшой демонстрационный датасет;
2. каждая запись содержит:
   - массив точек `points`;
   - глобальный класс объекта `class_id`;
   - пометку для каждой точки `seg_labels`;
3. один и тот же класс `PointCloudDataset` умеет работать в двух режимах:
   - `task="classification"`;
   - `task="segmentation"`.

### Какие библиотеки используются и зачем

- `numpy` — генерация облаков и сохранение данных в `NPZ`.
- `torch` — реализация класса `Dataset` и загрузчиков `DataLoader`.
- `matplotlib` — визуализация одной выборки с посегментной окраской.
- `pathlib` — удобная работа с файловой структурой.

### Готовый код для Google Colab

Основной код:

```python
from pathlib import Path
import numpy as np
import torch
from torch.utils.data import Dataset, DataLoader
import matplotlib.pyplot as plt


ROOT = Path("pointcloud_demo_dataset")
CLASS_NAMES = ["cube", "sphere", "cylinder"]
CLASS_TO_ID = {name: idx for idx, name in enumerate(CLASS_NAMES)}


def normalize_points(points):
    points = points - points.mean(axis=0, keepdims=True)
    scale = np.max(np.linalg.norm(points, axis=1))
    return points / (scale + 1e-8)


def rotation_matrix_z(angle):
    c = np.cos(angle)
    s = np.sin(angle)
    return np.array([
        [c, -s, 0.0],
        [s,  c, 0.0],
        [0.0, 0.0, 1.0],
    ], dtype=np.float32)


def sample_cube(n, rng):
    pts = rng.uniform(-0.5, 0.5, size=(n, 3))
    faces = rng.integers(0, 6, size=n)
    pts[faces == 0, 0] = -0.5
    pts[faces == 1, 0] = 0.5
    pts[faces == 2, 1] = -0.5
    pts[faces == 3, 1] = 0.5
    pts[faces == 4, 2] = -0.5
    pts[faces == 5, 2] = 0.5
    return pts.astype(np.float32)


def sample_sphere(n, rng):
    phi = rng.uniform(0, 2 * np.pi, n)
    cos_theta = rng.uniform(-1, 1, n)
    sin_theta = np.sqrt(1.0 - cos_theta ** 2)
    pts = 0.55 * np.column_stack([
        sin_theta * np.cos(phi),
        sin_theta * np.sin(phi),
        cos_theta,
    ])
    return pts.astype(np.float32)


def sample_cylinder(n, rng):
    pts = np.zeros((n, 3), dtype=np.float32)
    selector = rng.random(n)
    side = selector < 0.7
    top = (selector >= 0.7) & (selector < 0.85)
    bottom = selector >= 0.85
    angles = rng.uniform(0, 2 * np.pi, n)

    pts[side, 0] = 0.35 * np.cos(angles[side])
    pts[side, 1] = 0.35 * np.sin(angles[side])
    pts[side, 2] = rng.uniform(-0.6, 0.6, side.sum())

    r_top = 0.35 * np.sqrt(rng.random(top.sum()))
    pts[top, 0] = r_top * np.cos(angles[top])
    pts[top, 1] = r_top * np.sin(angles[top])
    pts[top, 2] = 0.6

    r_bottom = 0.35 * np.sqrt(rng.random(bottom.sum()))
    pts[bottom, 0] = r_bottom * np.cos(angles[bottom])
    pts[bottom, 1] = r_bottom * np.sin(angles[bottom])
    pts[bottom, 2] = -0.6
    return pts


def make_seg_labels(points):
    z = points[:, 2]
    return np.digitize(z, bins=[-0.15, 0.15]).astype(np.int64)


def generate_demo_dataset(root, train_count=40, val_count=10, test_count=10, points_per_sample=2048):
    generators = {
        "cube": sample_cube,
        "sphere": sample_sphere,
        "cylinder": sample_cylinder,
    }
    split_sizes = {"train": train_count, "val": val_count, "test": test_count}
    split_offsets = {"train": 0, "val": 10_000, "test": 20_000}

    if root.exists():
        has_files = any(root.glob("*/*.npz"))
        if has_files:
            return

    for split, count in split_sizes.items():
        split_dir = root / split
        split_dir.mkdir(parents=True, exist_ok=True)
        file_idx = 0

        for class_name, generator in generators.items():
            class_id = CLASS_TO_ID[class_name]
            for sample_idx in range(count):
                rng = np.random.default_rng(split_offsets[split] + class_id * 1000 + sample_idx)
                points = generator(points_per_sample, rng)
                angle = rng.uniform(0, 2 * np.pi)
                points = points @ rotation_matrix_z(angle).T
                points += rng.normal(0.0, 0.01, size=points.shape)
                seg_labels = make_seg_labels(points)

                np.savez_compressed(
                    split_dir / f"{class_name}_{file_idx:04d}.npz",
                    points=points.astype(np.float32),
                    class_id=np.int64(class_id),
                    seg_labels=seg_labels.astype(np.int64),
                )
                file_idx += 1


class PointCloudDataset(Dataset):
    def __init__(self, root, split="train", task="classification", num_points=1024, normalize=True, augment=False):
        self.root = Path(root)
        self.split = split
        self.task = task
        self.num_points = num_points
        self.normalize = normalize
        self.augment = augment
        self.files = sorted((self.root / split).glob("*.npz"))

        if not self.files:
            raise FileNotFoundError(f"В каталоге {self.root / split} нет файлов .npz")
        if self.task not in {"classification", "segmentation"}:
            raise ValueError("task должен быть 'classification' или 'segmentation'")

    def __len__(self):
        return len(self.files)

    def _augment_points(self, points):
        angle = np.random.uniform(0, 2 * np.pi)
        points = points @ rotation_matrix_z(angle).T
        points += np.random.normal(0.0, 0.005, size=points.shape)
        return points

    def __getitem__(self, index):
        data = np.load(self.files[index])
        points = data["points"].astype(np.float32)
        class_id = int(data["class_id"])
        seg_labels = data["seg_labels"].astype(np.int64)

        choice = np.random.choice(len(points), self.num_points, replace=len(points) < self.num_points)
        points = points[choice]
        seg_labels = seg_labels[choice]

        if self.normalize:
            points = normalize_points(points).astype(np.float32)
        if self.augment and self.split == "train":
            points = self._augment_points(points).astype(np.float32)

        points = torch.from_numpy(points)
        class_id = torch.tensor(class_id, dtype=torch.long)
        seg_labels = torch.from_numpy(seg_labels)

        if self.task == "classification":
            return points, class_id
        return points, seg_labels, class_id


def plot_sample(points, labels, title):
    palette = np.array(["#1f77b4", "#2ca02c", "#d62728"])
    fig = plt.figure(figsize=(5, 5))
    ax = fig.add_subplot(111, projection="3d")
    ax.scatter(points[:, 0], points[:, 1], points[:, 2], c=palette[labels], s=4)
    ax.set_title(title)
    ax.set_xlabel("X")
    ax.set_ylabel("Y")
    ax.set_zlabel("Z")
    ax.set_box_aspect((1, 1, 1))
    plt.tight_layout()
    plt.show()


generate_demo_dataset(ROOT)

train_cls = PointCloudDataset(ROOT, split="train", task="classification", num_points=1024, augment=True)
train_seg = PointCloudDataset(ROOT, split="train", task="segmentation", num_points=1024, augment=True)

cls_loader = DataLoader(train_cls, batch_size=16, shuffle=True)
seg_loader = DataLoader(train_seg, batch_size=8, shuffle=True)

cls_points, cls_targets = next(iter(cls_loader))
seg_points, seg_labels, seg_class_ids = next(iter(seg_loader))

print("Размер батча классификации:", cls_points.shape, cls_targets.shape)
print("Размер батча сегментации:", seg_points.shape, seg_labels.shape, seg_class_ids.shape)
print("Число train-файлов:", len(train_cls))
print("Число val-файлов:", len(PointCloudDataset(ROOT, split='val', task='classification')))
print("Число test-файлов:", len(PointCloudDataset(ROOT, split='test', task='classification')))

sample_points, sample_seg_labels, sample_class = train_seg[0]
print("Пример class_id:", sample_class.item(), "=>", CLASS_NAMES[sample_class.item()])
plot_sample(sample_points.numpy(), sample_seg_labels.numpy(), "Пример сегментационной разметки")
```

### Как запустить код и какой результат должен получиться

1. Выполнить ячейку с кодом.
2. Дождаться автоматической генерации учебного датасета.

Результат:

- создастся каталог `pointcloud_demo_dataset` со сплитами `train`, `val`, `test`;
- будут созданы `DataLoader` для двух режимов;
- в консоль выведутся размеры батчей;
- появится 3D-график с примером посегментной разметки точек.

### Краткие замечания об ограничениях, допущениях или возможных улучшениях

- Датасет синтетический, потому что в задании не указан формат реальных данных.
- В реальной задаче в запись обычно добавляют нормали, цвета, интенсивность отражения, индексы сцен и метаданные сенсора.
- Для крупных проектов стоит вынести логику чтения, нормализации, аугментаций и кэширования в отдельные модули.

## Весенний семестр

## Задание 4. Классификация облаков точек и оценка погрешности измерительной системы на основе PointNet

### Исходная формулировка

> Разработать программу классификации облаков точек, оценить погрешности измерительной системы, использующей обученную нейросеть архитектуры PointNet.

### Краткая цель и идея решения

В постановке нет готового датасета и нет описания конкретной измерительной системы. Поэтому в учебном варианте вводится разумное допущение:

- измерительная погрешность моделируется как добавление гауссовского шума к координатам точек;
- классифицируются синтетические облака четырёх классов: `cube`, `sphere`, `cylinder`, `cone`;
- используется упрощённая учебная реализация PointNet: общая `MLP` над точками, симметричный `max pooling`, далее полносвязный классификатор.

Идея решения:

1. Сгенерировать учебный набор облаков.
2. Обучить классификатор PointNet.
3. Проверить качество на тесте.
4. Прогнать модель через серию уровней шума.
5. Оценить, при каком уровне шумов точность становится недопустимой.

### Какие библиотеки используются и зачем

- `numpy` — генерация синтетических облаков и управление шумом.
- `torch` — реализация сети PointNet и обучения.
- `sklearn` — матрица ошибок и текстовый отчёт по качеству.
- `matplotlib` — графики обучения и устойчивости к шуму.

### Готовый код для Google Colab

Основной код:

```python
import math
import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.utils.data import Dataset, DataLoader
from sklearn.metrics import confusion_matrix, classification_report
import matplotlib.pyplot as plt


torch.manual_seed(42)
np.random.seed(42)

CLASS_NAMES = ["cube", "sphere", "cylinder", "cone"]


def normalize_points(points):
    points = points - points.mean(axis=0, keepdims=True)
    scale = np.max(np.linalg.norm(points, axis=1))
    return points / (scale + 1e-8)


def random_rotation_matrix(rng):
    az = rng.uniform(0, 2 * np.pi)
    el = rng.uniform(-np.pi / 8, np.pi / 8)

    cz, sz = np.cos(az), np.sin(az)
    cx, sx = np.cos(el), np.sin(el)

    rz = np.array([
        [cz, -sz, 0.0],
        [sz,  cz, 0.0],
        [0.0, 0.0, 1.0],
    ])
    rx = np.array([
        [1.0, 0.0, 0.0],
        [0.0, cx, -sx],
        [0.0, sx,  cx],
    ])
    return (rz @ rx).astype(np.float32)


def sample_cube_surface(n, rng):
    pts = rng.uniform(-0.5, 0.5, size=(n, 3))
    faces = rng.integers(0, 6, size=n)
    pts[faces == 0, 0] = -0.5
    pts[faces == 1, 0] = 0.5
    pts[faces == 2, 1] = -0.5
    pts[faces == 3, 1] = 0.5
    pts[faces == 4, 2] = -0.5
    pts[faces == 5, 2] = 0.5
    return pts.astype(np.float32)


def sample_sphere_surface(n, rng):
    phi = rng.uniform(0, 2 * np.pi, n)
    cos_theta = rng.uniform(-1, 1, n)
    sin_theta = np.sqrt(1.0 - cos_theta ** 2)
    pts = 0.6 * np.column_stack([
        sin_theta * np.cos(phi),
        sin_theta * np.sin(phi),
        cos_theta,
    ])
    return pts.astype(np.float32)


def sample_cylinder_surface(n, rng):
    pts = np.zeros((n, 3), dtype=np.float32)
    selector = rng.random(n)
    side = selector < 0.7
    top = (selector >= 0.7) & (selector < 0.85)
    bottom = selector >= 0.85
    angles = rng.uniform(0, 2 * np.pi, n)

    pts[side, 0] = 0.45 * np.cos(angles[side])
    pts[side, 1] = 0.45 * np.sin(angles[side])
    pts[side, 2] = rng.uniform(-0.6, 0.6, side.sum())

    r_top = 0.45 * np.sqrt(rng.random(top.sum()))
    pts[top, 0] = r_top * np.cos(angles[top])
    pts[top, 1] = r_top * np.sin(angles[top])
    pts[top, 2] = 0.6

    r_bottom = 0.45 * np.sqrt(rng.random(bottom.sum()))
    pts[bottom, 0] = r_bottom * np.cos(angles[bottom])
    pts[bottom, 1] = r_bottom * np.sin(angles[bottom])
    pts[bottom, 2] = -0.6
    return pts


def sample_cone_surface(n, rng):
    pts = np.zeros((n, 3), dtype=np.float32)
    selector = rng.random(n)
    side = selector < 0.8
    base = ~side
    angles = rng.uniform(0, 2 * np.pi, n)

    h = rng.uniform(0.0, 1.0, side.sum())
    radius = 0.5 * (1.0 - h)
    pts[side, 0] = radius * np.cos(angles[side])
    pts[side, 1] = radius * np.sin(angles[side])
    pts[side, 2] = 1.2 * h - 0.6

    r_base = 0.5 * np.sqrt(rng.random(base.sum()))
    pts[base, 0] = r_base * np.cos(angles[base])
    pts[base, 1] = r_base * np.sin(angles[base])
    pts[base, 2] = -0.6
    return pts


SHAPE_GENERATORS = {
    0: sample_cube_surface,
    1: sample_sphere_surface,
    2: sample_cylinder_surface,
    3: sample_cone_surface,
}


class SyntheticShapeDataset(Dataset):
    def __init__(self, samples_per_class, num_points=512, noise_sigma=0.0, split="train", seed=0):
        self.samples_per_class = samples_per_class
        self.num_points = num_points
        self.noise_sigma = noise_sigma
        self.split = split
        self.seed = seed
        self.samples = [
            (class_id, sample_idx)
            for class_id in range(len(CLASS_NAMES))
            for sample_idx in range(samples_per_class)
        ]
        self.split_offset = {"train": 0, "val": 100_000, "test": 200_000}[split]

    def __len__(self):
        return len(self.samples)

    def __getitem__(self, index):
        class_id, sample_idx = self.samples[index]
        rng = np.random.default_rng(self.seed + self.split_offset + class_id * 10_000 + sample_idx)

        points = SHAPE_GENERATORS[class_id](self.num_points, rng)
        points = points @ random_rotation_matrix(rng).T
        points *= rng.uniform(0.9, 1.1)
        points += rng.normal(0.0, self.noise_sigma, size=points.shape)
        points = normalize_points(points).astype(np.float32)

        return torch.from_numpy(points), torch.tensor(class_id, dtype=torch.long)


class MiniPointNet(nn.Module):
    def __init__(self, num_classes):
        super().__init__()
        self.backbone = nn.Sequential(
            nn.Conv1d(3, 64, 1),
            nn.BatchNorm1d(64),
            nn.ReLU(),
            nn.Conv1d(64, 128, 1),
            nn.BatchNorm1d(128),
            nn.ReLU(),
            nn.Conv1d(128, 256, 1),
            nn.BatchNorm1d(256),
            nn.ReLU(),
            nn.Conv1d(256, 512, 1),
            nn.BatchNorm1d(512),
            nn.ReLU(),
        )
        self.classifier = nn.Sequential(
            nn.Linear(512, 256),
            nn.BatchNorm1d(256),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(256, 128),
            nn.BatchNorm1d(128),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(128, num_classes),
        )

    def forward(self, x):
        x = x.transpose(1, 2)
        x = self.backbone(x)
        x = torch.max(x, dim=2).values
        return self.classifier(x)


def run_epoch(model, loader, optimizer=None):
    training = optimizer is not None
    model.train(training)

    total_loss = 0.0
    total_correct = 0
    total_items = 0
    y_true = []
    y_pred = []

    for points, targets in loader:
        points = points.to(device)
        targets = targets.to(device)

        logits = model(points)
        loss = F.cross_entropy(logits, targets)

        if training:
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()

        preds = logits.argmax(dim=1)
        total_loss += loss.item() * len(points)
        total_correct += (preds == targets).sum().item()
        total_items += len(points)
        y_true.extend(targets.cpu().numpy().tolist())
        y_pred.extend(preds.cpu().numpy().tolist())

    return {
        "loss": total_loss / total_items,
        "acc": total_correct / total_items,
        "y_true": np.array(y_true),
        "y_pred": np.array(y_pred),
    }


def plot_confusion(cm, class_names, title):
    fig, ax = plt.subplots(figsize=(6, 5))
    im = ax.imshow(cm, cmap="Blues")
    ax.set_xticks(range(len(class_names)))
    ax.set_yticks(range(len(class_names)))
    ax.set_xticklabels(class_names, rotation=45, ha="right")
    ax.set_yticklabels(class_names)
    ax.set_title(title)
    ax.set_xlabel("Предсказанный класс")
    ax.set_ylabel("Истинный класс")

    for i in range(cm.shape[0]):
        for j in range(cm.shape[1]):
            ax.text(j, i, cm[i, j], ha="center", va="center", color="black")

    fig.colorbar(im, ax=ax)
    plt.tight_layout()
    plt.show()


device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
print("Устройство:", device)

train_dataset = SyntheticShapeDataset(samples_per_class=180, num_points=512, noise_sigma=0.01, split="train", seed=123)
val_dataset = SyntheticShapeDataset(samples_per_class=40, num_points=512, noise_sigma=0.01, split="val", seed=123)
test_dataset = SyntheticShapeDataset(samples_per_class=60, num_points=512, noise_sigma=0.01, split="test", seed=123)

train_loader = DataLoader(train_dataset, batch_size=32, shuffle=True, num_workers=0)
val_loader = DataLoader(val_dataset, batch_size=64, shuffle=False, num_workers=0)
test_loader = DataLoader(test_dataset, batch_size=64, shuffle=False, num_workers=0)

model = MiniPointNet(num_classes=len(CLASS_NAMES)).to(device)
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3, weight_decay=1e-4)
scheduler = torch.optim.lr_scheduler.StepLR(optimizer, step_size=8, gamma=0.5)

history = {"train_loss": [], "train_acc": [], "val_loss": [], "val_acc": []}

epochs = 12
for epoch in range(1, epochs + 1):
    train_metrics = run_epoch(model, train_loader, optimizer=optimizer)
    val_metrics = run_epoch(model, val_loader, optimizer=None)
    scheduler.step()

    history["train_loss"].append(train_metrics["loss"])
    history["train_acc"].append(train_metrics["acc"])
    history["val_loss"].append(val_metrics["loss"])
    history["val_acc"].append(val_metrics["acc"])

    print(
        f"Epoch {epoch:02d} | "
        f"train_loss={train_metrics['loss']:.4f} train_acc={train_metrics['acc']:.4f} | "
        f"val_loss={val_metrics['loss']:.4f} val_acc={val_metrics['acc']:.4f}"
    )

test_metrics = run_epoch(model, test_loader, optimizer=None)
print(f"\nТочность на тесте: {test_metrics['acc']:.4f}")
print("\nОтчёт по классам:")
print(classification_report(test_metrics["y_true"], test_metrics["y_pred"], target_names=CLASS_NAMES, digits=3))

cm = confusion_matrix(test_metrics["y_true"], test_metrics["y_pred"], labels=range(len(CLASS_NAMES)))
plot_confusion(cm, CLASS_NAMES, "Матрица ошибок на тестовой выборке")

fig, axes = plt.subplots(1, 2, figsize=(12, 4))
axes[0].plot(history["train_loss"], label="train")
axes[0].plot(history["val_loss"], label="val")
axes[0].set_title("Функция потерь")
axes[0].set_xlabel("Epoch")
axes[0].legend()

axes[1].plot(history["train_acc"], label="train")
axes[1].plot(history["val_acc"], label="val")
axes[1].set_title("Точность")
axes[1].set_xlabel("Epoch")
axes[1].legend()
plt.tight_layout()
plt.show()

noise_levels = [0.0, 0.005, 0.01, 0.02, 0.03, 0.05]
noise_results = []

for sigma in noise_levels:
    noisy_dataset = SyntheticShapeDataset(
        samples_per_class=60,
        num_points=512,
        noise_sigma=sigma,
        split="test",
        seed=999,
    )
    noisy_loader = DataLoader(noisy_dataset, batch_size=64, shuffle=False, num_workers=0)
    metrics = run_epoch(model, noisy_loader, optimizer=None)
    rmse_xyz = math.sqrt(3.0) * sigma
    noise_results.append((sigma, rmse_xyz, metrics["acc"]))

print("\nУстойчивость к шуму:")
for sigma, rmse_xyz, acc in noise_results:
    print(f"sigma={sigma:.3f} | RMSE_xyz={rmse_xyz:.4f} | accuracy={acc:.4f}")

target_accuracy = 0.90
acceptable = [item for item in noise_results if item[2] >= target_accuracy]
if acceptable:
    best_sigma, best_rmse, _ = acceptable[-1]
    print(f"\nПри пороге accuracy >= {target_accuracy:.2f} допустимый уровень шума: sigma <= {best_sigma:.3f}")
    print(f"Это соответствует интегральной RMS-ошибке по XYZ около {best_rmse:.4f}")
else:
    print(f"\nДля порога accuracy >= {target_accuracy:.2f} ни один уровень шума не прошёл критерий.")

plt.figure(figsize=(7, 4))
plt.plot([x[0] for x in noise_results], [x[2] for x in noise_results], marker="o")
plt.axhline(target_accuracy, color="red", linestyle="--", label=f"Порог {target_accuracy:.2f}")
plt.xlabel("Стандартное отклонение шума sigma")
plt.ylabel("Accuracy")
plt.title("Чувствительность PointNet к координатному шуму")
plt.grid(True)
plt.legend()
plt.show()
```

### Как запустить код и какой результат должен получиться

1. Желательно включить в Colab аппаратный ускоритель `GPU`.
2. Выполнить ячейку с кодом целиком.
3. Дождаться окончания обучения.

Ожидаемый результат:

- сеть обучится распознавать 4 класса синтетических облаков;
- в консоль будет выведена точность на тесте и отчёт по классам;
- появится матрица ошибок;
- появятся графики обучения;
- будет получена зависимость точности от уровня координатного шума.

### Краткие замечания об ограничениях, допущениях или возможных улучшениях

- Здесь PointNet реализован в учебном упрощённом варианте без `T-Net` и без feature transform.
- Погрешность измерительной системы моделируется только гауссовским шумом координат. В реальности нужно учитывать систематические смещения, пропуски точек, неоднородную плотность, окклюзии и выбросы.
- Для реальных инженерных выводов модель следует обучать на данных конкретного сенсора и отдельно валидировать на реальных измерениях.

## Задание 5. Сегментация облака точек на основе PoinNet++

### Исходная формулировка

> Разработать программу сегментации облака точек на основе архитектуры PoinNet++, оценить допустимую ошибку разделения классов и другие метрики.

### Краткая цель и идея решения

В исходной формулировке не задан датасет, поэтому в учебном решении используется синтетический набор сцен с четырьмя семантическими классами:

- `ground`;
- `box`;
- `sphere`;
- `cylinder`.

Сеть строится по мотивам PointNet++:

1. иерархическая выборка опорных точек `FPS`;
2. локальная группировка ближайших соседей `kNN`;
3. извлечение локальных признаков `shared MLP`;
4. обратное распространение признаков к исходным точкам `feature propagation`;
5. посегментная классификация всех точек сцены.

### Какие библиотеки используются и зачем

- `numpy` — генерация синтетических сцен.
- `torch` — реализация упрощённого PointNet++ и обучение.
- `sklearn` — матрица ошибок.
- `matplotlib` — визуализация кривых обучения и сравнение истинной/предсказанной разметки.

### Готовый код для Google Colab

Основной код:

```python
import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.utils.data import Dataset, DataLoader
from sklearn.metrics import confusion_matrix
import matplotlib.pyplot as plt


torch.manual_seed(7)
np.random.seed(7)

CLASS_NAMES = ["ground", "box", "sphere", "cylinder"]


def sample_plane(n, rng):
    x = rng.uniform(-1.2, 1.2, n)
    y = rng.uniform(-1.2, 1.2, n)
    z = rng.normal(0.0, 0.003, n)
    return np.column_stack([x, y, z]).astype(np.float32)


def sample_box_surface(n, center, size, rng):
    sx, sy, sz = size
    pts = rng.uniform(-0.5, 0.5, size=(n, 3))
    faces = rng.integers(0, 6, size=n)
    pts[:, 0] *= sx
    pts[:, 1] *= sy
    pts[:, 2] *= sz
    pts[faces == 0, 0] = -sx / 2
    pts[faces == 1, 0] =  sx / 2
    pts[faces == 2, 1] = -sy / 2
    pts[faces == 3, 1] =  sy / 2
    pts[faces == 4, 2] = -sz / 2
    pts[faces == 5, 2] =  sz / 2
    pts += center
    return pts.astype(np.float32)


def sample_sphere_surface(n, center, radius, rng):
    phi = rng.uniform(0, 2 * np.pi, n)
    cos_theta = rng.uniform(-1, 1, n)
    sin_theta = np.sqrt(1.0 - cos_theta ** 2)
    pts = np.column_stack([
        radius * sin_theta * np.cos(phi),
        radius * sin_theta * np.sin(phi),
        radius * cos_theta,
    ])
    pts += center
    return pts.astype(np.float32)


def sample_cylinder_surface(n, center, radius, height, rng):
    pts = np.zeros((n, 3), dtype=np.float32)
    selector = rng.random(n)
    side = selector < 0.7
    top = (selector >= 0.7) & (selector < 0.85)
    bottom = selector >= 0.85
    angles = rng.uniform(0, 2 * np.pi, n)

    pts[side, 0] = radius * np.cos(angles[side])
    pts[side, 1] = radius * np.sin(angles[side])
    pts[side, 2] = rng.uniform(-height / 2, height / 2, side.sum())

    r_top = radius * np.sqrt(rng.random(top.sum()))
    pts[top, 0] = r_top * np.cos(angles[top])
    pts[top, 1] = r_top * np.sin(angles[top])
    pts[top, 2] = height / 2

    r_bottom = radius * np.sqrt(rng.random(bottom.sum()))
    pts[bottom, 0] = r_bottom * np.cos(angles[bottom])
    pts[bottom, 1] = r_bottom * np.sin(angles[bottom])
    pts[bottom, 2] = -height / 2

    pts += center
    return pts


def make_scene(num_points=768, noise_sigma=0.005, rng=None):
    if rng is None:
        rng = np.random.default_rng()

    counts = {"ground": 256, "box": 176, "sphere": 168, "cylinder": 168}

    ground = sample_plane(counts["ground"], rng)

    box_center = np.array([rng.uniform(-0.6, -0.1), rng.uniform(-0.1, 0.6), 0.28], dtype=np.float32)
    sphere_center = np.array([rng.uniform(0.2, 0.8), rng.uniform(-0.7, -0.1), 0.32], dtype=np.float32)
    cyl_center = np.array([rng.uniform(-0.2, 0.7), rng.uniform(-0.8, 0.2), 0.35], dtype=np.float32)

    box = sample_box_surface(counts["box"], box_center, size=(0.48, 0.38, 0.52), rng=rng)
    sphere = sample_sphere_surface(counts["sphere"], sphere_center, radius=0.27, rng=rng)
    cylinder = sample_cylinder_surface(counts["cylinder"], cyl_center, radius=0.22, height=0.72, rng=rng)

    points = np.vstack([ground, box, sphere, cylinder])
    labels = np.concatenate([
        np.zeros(counts["ground"], dtype=np.int64),
        np.ones(counts["box"], dtype=np.int64),
        np.full(counts["sphere"], 2, dtype=np.int64),
        np.full(counts["cylinder"], 3, dtype=np.int64),
    ])

    points += rng.normal(0.0, noise_sigma, size=points.shape)
    perm = rng.permutation(len(points))
    points = points[perm]
    labels = labels[perm]

    mins = points.min(axis=0, keepdims=True)
    maxs = points.max(axis=0, keepdims=True)
    points = 2.0 * (points - mins) / (maxs - mins + 1e-8) - 1.0
    return points.astype(np.float32), labels.astype(np.int64)


class SyntheticSceneDataset(Dataset):
    def __init__(self, n_scenes, num_points=768, noise_sigma=0.005, split="train", seed=0):
        self.n_scenes = n_scenes
        self.num_points = num_points
        self.noise_sigma = noise_sigma
        self.split = split
        self.seed = seed
        self.split_offset = {"train": 0, "val": 100_000, "test": 200_000}[split]

    def __len__(self):
        return self.n_scenes

    def __getitem__(self, index):
        rng = np.random.default_rng(self.seed + self.split_offset + index)
        points, labels = make_scene(num_points=self.num_points, noise_sigma=self.noise_sigma, rng=rng)
        return torch.from_numpy(points), torch.from_numpy(labels)


def square_distance(src, dst):
    return (
        torch.sum(src ** 2, dim=-1, keepdim=True)
        - 2 * torch.matmul(src, dst.transpose(1, 2))
        + torch.sum(dst ** 2, dim=-1).unsqueeze(1)
    )


def index_points(points, idx):
    device = points.device
    batch_size = points.shape[0]
    view_shape = list(idx.shape)
    view_shape[1:] = [1] * (len(view_shape) - 1)
    repeat_shape = list(idx.shape)
    repeat_shape[0] = 1
    batch_indices = torch.arange(batch_size, dtype=torch.long, device=device).view(view_shape).repeat(repeat_shape)
    return points[batch_indices, idx, :]


def farthest_point_sample(xyz, npoint):
    device = xyz.device
    batch_size, num_points, _ = xyz.shape
    centroids = torch.zeros(batch_size, npoint, dtype=torch.long, device=device)
    distance = torch.ones(batch_size, num_points, device=device) * 1e10
    farthest = torch.randint(0, num_points, (batch_size,), dtype=torch.long, device=device)
    batch_indices = torch.arange(batch_size, dtype=torch.long, device=device)

    for i in range(npoint):
        centroids[:, i] = farthest
        centroid = xyz[batch_indices, farthest, :].view(batch_size, 1, 3)
        dist = torch.sum((xyz - centroid) ** 2, dim=-1)
        distance = torch.minimum(distance, dist)
        farthest = torch.max(distance, dim=-1).indices
    return centroids


def knn_point(k, xyz, new_xyz):
    distances = square_distance(new_xyz, xyz)
    return torch.topk(distances, k=k, dim=-1, largest=False).indices


def sample_and_group(npoint, k, xyz, points):
    fps_idx = farthest_point_sample(xyz, npoint)
    new_xyz = index_points(xyz, fps_idx)
    group_idx = knn_point(k, xyz, new_xyz)
    grouped_xyz = index_points(xyz, group_idx)
    grouped_xyz_norm = grouped_xyz - new_xyz.unsqueeze(2)

    if points is not None:
        grouped_points = index_points(points, group_idx)
        new_points = torch.cat([grouped_xyz_norm, grouped_points], dim=-1)
    else:
        new_points = grouped_xyz_norm

    return new_xyz, new_points


def three_nn(unknown, known):
    distances = square_distance(unknown, known)
    dists, idx = torch.topk(distances, k=3, dim=-1, largest=False)
    return torch.sqrt(dists + 1e-8), idx


class SetAbstraction(nn.Module):
    def __init__(self, npoint, k, in_channel, mlp):
        super().__init__()
        self.npoint = npoint
        self.k = k
        last_channel = in_channel + 3
        self.convs = nn.ModuleList()
        self.bns = nn.ModuleList()
        for out_channel in mlp:
            self.convs.append(nn.Conv2d(last_channel, out_channel, 1))
            self.bns.append(nn.BatchNorm2d(out_channel))
            last_channel = out_channel

    def forward(self, xyz, points):
        new_xyz, new_points = sample_and_group(self.npoint, self.k, xyz, points)
        new_points = new_points.permute(0, 3, 2, 1)
        for conv, bn in zip(self.convs, self.bns):
            new_points = F.relu(bn(conv(new_points)))
        new_points = torch.max(new_points, dim=2).values
        new_points = new_points.transpose(1, 2)
        return new_xyz, new_points


class FeaturePropagation(nn.Module):
    def __init__(self, in_channel, mlp):
        super().__init__()
        self.convs = nn.ModuleList()
        self.bns = nn.ModuleList()
        last_channel = in_channel
        for out_channel in mlp:
            self.convs.append(nn.Conv2d(last_channel, out_channel, 1))
            self.bns.append(nn.BatchNorm2d(out_channel))
            last_channel = out_channel

    def forward(self, xyz1, xyz2, points1, points2):
        if xyz2.shape[1] == 1:
            interpolated = points2.repeat(1, xyz1.shape[1], 1)
        else:
            dists, idx = three_nn(xyz1, xyz2)
            inv_dist = 1.0 / (dists + 1e-8)
            norm = torch.sum(inv_dist, dim=2, keepdim=True)
            weight = inv_dist / norm
            interpolated = torch.sum(index_points(points2, idx) * weight.unsqueeze(-1), dim=2)

        if points1 is not None:
            new_points = torch.cat([points1, interpolated], dim=-1)
        else:
            new_points = interpolated

        new_points = new_points.transpose(1, 2).unsqueeze(-1)
        for conv, bn in zip(self.convs, self.bns):
            new_points = F.relu(bn(conv(new_points)))
        new_points = new_points.squeeze(-1).transpose(1, 2)
        return new_points


class MiniPointNet2Seg(nn.Module):
    def __init__(self, num_classes):
        super().__init__()
        self.sa1 = SetAbstraction(npoint=128, k=24, in_channel=0, mlp=[64, 64, 128])
        self.sa2 = SetAbstraction(npoint=32, k=24, in_channel=128, mlp=[128, 128, 256])
        self.fp2 = FeaturePropagation(in_channel=128 + 256, mlp=[256, 128])
        self.fp1 = FeaturePropagation(in_channel=3 + 128, mlp=[128, 128, 64])
        self.head = nn.Sequential(
            nn.Conv1d(64, 64, 1),
            nn.BatchNorm1d(64),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Conv1d(64, num_classes, 1),
        )

    def forward(self, xyz):
        l0_xyz = xyz
        l0_points = None

        l1_xyz, l1_points = self.sa1(l0_xyz, l0_points)
        l2_xyz, l2_points = self.sa2(l1_xyz, l1_points)

        l1_points = self.fp2(l1_xyz, l2_xyz, l1_points, l2_points)
        l0_points = self.fp1(l0_xyz, l1_xyz, l0_xyz, l1_points)

        x = l0_points.transpose(1, 2)
        x = self.head(x)
        return x.transpose(1, 2)


def segmentation_metrics(y_true, y_pred, num_classes):
    cm = confusion_matrix(y_true, y_pred, labels=list(range(num_classes)))
    overall_acc = np.trace(cm) / np.sum(cm)

    per_class_acc = []
    per_class_iou = []
    for cls in range(num_classes):
        tp = cm[cls, cls]
        fn = cm[cls, :].sum() - tp
        fp = cm[:, cls].sum() - tp

        acc = tp / max(cm[cls, :].sum(), 1)
        iou = tp / max(tp + fp + fn, 1)
        per_class_acc.append(acc)
        per_class_iou.append(iou)

    mean_iou = float(np.mean(per_class_iou))
    return cm, overall_acc, per_class_acc, per_class_iou, mean_iou


def run_epoch(model, loader, optimizer=None):
    training = optimizer is not None
    model.train(training)

    total_loss = 0.0
    total_points = 0
    y_true = []
    y_pred = []

    for points, labels in loader:
        points = points.to(device)
        labels = labels.to(device)
        logits = model(points)
        loss = F.cross_entropy(logits.reshape(-1, logits.shape[-1]), labels.reshape(-1))

        if training:
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()

        preds = logits.argmax(dim=-1)
        total_loss += loss.item() * labels.numel()
        total_points += labels.numel()
        y_true.extend(labels.cpu().numpy().reshape(-1).tolist())
        y_pred.extend(preds.cpu().numpy().reshape(-1).tolist())

    cm, overall_acc, per_class_acc, per_class_iou, mean_iou = segmentation_metrics(
        np.array(y_true), np.array(y_pred), len(CLASS_NAMES)
    )

    return {
        "loss": total_loss / total_points,
        "overall_acc": overall_acc,
        "per_class_acc": per_class_acc,
        "per_class_iou": per_class_iou,
        "mean_iou": mean_iou,
        "cm": cm,
    }


def plot_confusion(cm, class_names, title):
    fig, ax = plt.subplots(figsize=(6, 5))
    im = ax.imshow(cm, cmap="Blues")
    ax.set_xticks(range(len(class_names)))
    ax.set_yticks(range(len(class_names)))
    ax.set_xticklabels(class_names, rotation=45, ha="right")
    ax.set_yticklabels(class_names)
    ax.set_title(title)
    ax.set_xlabel("Предсказанный класс")
    ax.set_ylabel("Истинный класс")

    for i in range(cm.shape[0]):
        for j in range(cm.shape[1]):
            ax.text(j, i, cm[i, j], ha="center", va="center", color="black")

    fig.colorbar(im, ax=ax)
    plt.tight_layout()
    plt.show()


def plot_segmentation(points, labels, title, ax):
    palette = np.array(["#808080", "#1f77b4", "#d62728", "#2ca02c"])
    ax.scatter(points[:, 0], points[:, 1], points[:, 2], c=palette[labels], s=4)
    ax.set_title(title)
    ax.set_xlabel("X")
    ax.set_ylabel("Y")
    ax.set_zlabel("Z")
    ax.set_box_aspect((1, 1, 0.8))


device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
print("Устройство:", device)

train_dataset = SyntheticSceneDataset(n_scenes=100, num_points=768, noise_sigma=0.005, split="train", seed=321)
val_dataset = SyntheticSceneDataset(n_scenes=20, num_points=768, noise_sigma=0.005, split="val", seed=321)
test_dataset = SyntheticSceneDataset(n_scenes=20, num_points=768, noise_sigma=0.005, split="test", seed=321)

train_loader = DataLoader(train_dataset, batch_size=8, shuffle=True, num_workers=0)
val_loader = DataLoader(val_dataset, batch_size=8, shuffle=False, num_workers=0)
test_loader = DataLoader(test_dataset, batch_size=8, shuffle=False, num_workers=0)

model = MiniPointNet2Seg(num_classes=len(CLASS_NAMES)).to(device)
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3, weight_decay=1e-4)

history = {"train_loss": [], "val_loss": [], "train_miou": [], "val_miou": []}

epochs = 10
for epoch in range(1, epochs + 1):
    train_metrics = run_epoch(model, train_loader, optimizer=optimizer)
    val_metrics = run_epoch(model, val_loader, optimizer=None)

    history["train_loss"].append(train_metrics["loss"])
    history["val_loss"].append(val_metrics["loss"])
    history["train_miou"].append(train_metrics["mean_iou"])
    history["val_miou"].append(val_metrics["mean_iou"])

    print(
        f"Epoch {epoch:02d} | "
        f"train_loss={train_metrics['loss']:.4f} train_mIoU={train_metrics['mean_iou']:.4f} | "
        f"val_loss={val_metrics['loss']:.4f} val_mIoU={val_metrics['mean_iou']:.4f}"
    )

test_metrics = run_epoch(model, test_loader, optimizer=None)
print(f"\nOverall accuracy: {test_metrics['overall_acc']:.4f}")
print(f"Mean IoU: {test_metrics['mean_iou']:.4f}")

for cls_name, acc, iou in zip(CLASS_NAMES, test_metrics["per_class_acc"], test_metrics["per_class_iou"]):
    print(f"{cls_name:>8s} | class_acc={acc:.4f} | IoU={iou:.4f}")

point_error = 1.0 - test_metrics["overall_acc"]
acceptable_limit = 0.10
print(f"\nТочечная ошибка разделения: {point_error:.4f}")
print(f"Критерий допустимости (ошибка <= {acceptable_limit:.2f}): {point_error <= acceptable_limit}")

plot_confusion(test_metrics["cm"], CLASS_NAMES, "Матрица ошибок сегментации")

fig, axes = plt.subplots(1, 2, figsize=(12, 4))
axes[0].plot(history["train_loss"], label="train")
axes[0].plot(history["val_loss"], label="val")
axes[0].set_title("Функция потерь")
axes[0].set_xlabel("Epoch")
axes[0].legend()

axes[1].plot(history["train_miou"], label="train")
axes[1].plot(history["val_miou"], label="val")
axes[1].set_title("Mean IoU")
axes[1].set_xlabel("Epoch")
axes[1].legend()
plt.tight_layout()
plt.show()

sample_points, sample_labels = test_dataset[0]
model.eval()
with torch.no_grad():
    sample_logits = model(sample_points.unsqueeze(0).to(device))
    sample_pred = sample_logits.argmax(dim=-1).squeeze(0).cpu().numpy()

fig = plt.figure(figsize=(12, 5))
ax1 = fig.add_subplot(121, projection="3d")
ax2 = fig.add_subplot(122, projection="3d")
plot_segmentation(sample_points.numpy(), sample_labels.numpy(), "Истинная разметка", ax1)
plot_segmentation(sample_points.numpy(), sample_pred, "Предсказание сети", ax2)
plt.tight_layout()
plt.show()
```

### Как запустить код и какой результат должен получиться

1. В Colab желательно выбрать `GPU`.
2. Выполнить весь код одним блоком или по частям сверху вниз.
3. Дождаться окончания обучения.

Результат:

- будет обучена учебная версия сегментационной сети по мотивам PointNet++;
- в консоль будут выведены `overall accuracy`, `per-class accuracy`, `IoU` и `mean IoU`;
- будет рассчитана точечная ошибка разделения классов;
- появится матрица ошибок;
- будет показан пример истинной и предсказанной сегментации.

### Краткие замечания об ограничениях, допущениях или возможных улучшениях

- В формулировке задания есть опечатка `PoinNet++`; в решении используется учебная реализация по мотивам `PointNet++`.
- Архитектура упрощена: вместо радиусного поиска применяется `kNN`, а набор сцен синтетический.
- Для реальных проектов стоит использовать полноценные датасеты `ShapeNet Part`, `S3DIS`, `ScanNet` или данные собственного сенсора.
- Для честной оценки допустимой ошибки полезно дополнительно считать `precision`, `recall`, `F1-score`, а также отдельно анализировать ошибки на границах классов.

## Задание 6. Реферат по методу Ball Pivoting Algorithm

### Исходная формулировка

> Подготовить реферат на тему «Поверхностная реконструкция на основе метода «Ball Pivoting Algorithm (BPA)»».

### Подробный план реферата

1. Введение.
2. Постановка задачи поверхностной реконструкции по облаку точек.
3. Предпосылки применения BPA и требования к входным данным.
4. Геометрическая идея метода Ball Pivoting Algorithm.
5. Пошаговый алгоритм работы BPA.
6. Выбор радиуса шара и влияние параметров на результат.
7. Оценка качества реконструкции.
8. Преимущества и недостатки BPA.
9. Сравнение BPA с альтернативными методами реконструкции.
10. Практические области применения.
11. Заключение.

### Ключевые тезисы по каждому разделу

**1. Введение**

- Поверхностная реконструкция нужна для перехода от неструктурированного облака точек к полигональной модели.
- Задача возникает в 3D-сканировании, реверс-инжиниринге, медицине, робототехнике и культурном наследии.
- BPA — один из классических методов локальной геометрической реконструкции.

**2. Постановка задачи**

- На вход подаётся набор точек, являющийся дискретным представлением поверхности объекта.
- Требуется восстановить треугольную сетку, максимально согласованную с геометрией исходной поверхности.
- Важны топологическая связность, локальная точность и устойчивость к шуму.

**3. Предпосылки применения BPA**

- Метод предполагает, что точки достаточно плотно и более или менее равномерно покрывают поверхность.
- Для качественной работы обычно нужны нормали или хотя бы корректно ориентированные локальные направления.
- Если в данных есть большие разрывы, дыры, сильный шум или резкая неравномерность плотности, качество BPA заметно падает.

**4. Геометрическая идея BPA**

- Воображаемый шар фиксированного радиуса касается трёх точек одновременно.
- Если шар касается трёх точек и не содержит внутри других точек, эти точки могут образовать треугольник поверхности.
- Далее шар "перекатывается" вокруг рёбер уже построенных треугольников и ищет новые точки касания.

**5. Пошаговый алгоритм**

- Найти стартовый треугольник, для которого допустимо касание шара.
- Добавить его в сетку и поместить рёбра в очередь активной границы.
- Для каждого активного ребра искать новую точку, с которой шар может образовать следующий треугольник.
- Повторять процесс, пока есть доступные активные рёбра.
- При необходимости запускать алгоритм с несколькими радиусами, если плотность выборки неоднородна.

**6. Выбор радиуса шара**

- Слишком маленький радиус приводит к разрывам и недовосстановленным участкам.
- Слишком большой радиус сглаживает детали и может соединять геометрически несвязанные части.
- На практике радиус часто выбирают на основе статистики расстояний до ближайших соседей.
- Набор из нескольких радиусов обычно работает надёжнее одного фиксированного значения.

**7. Оценка качества реконструкции**

- Оценивают полноту сетки, число дыр, корректность нормалей, отсутствие самопересечений.
- При наличии эталонной поверхности используют метрики расстояния `Chamfer Distance`, `Hausdorff Distance`, отклонение нормалей.
- Если сетка нужна для инженерного применения, дополнительно смотрят на водонепроницаемость и качество треугольников.

**8. Преимущества и недостатки**

- Достоинства: простая геометрическая идея, хорошая интерполяция поверхности, контроль локальной структуры.
- Недостатки: чувствительность к выбору радиуса, шуму и неоднородной плотности выборки.
- BPA не является универсальным решением и хуже работает на очень неполных или сильно зашумлённых данных.

**9. Сравнение с другими методами**

- По сравнению с Poisson-реконструкцией BPA обычно лучше интерполирует исходные точки, но хуже закрывает крупные пробелы.
- По сравнению с `alpha shapes` BPA больше ориентирован на локальное пошаговое построение поверхности.
- По сравнению с методами на базе Delaunay-триангуляции BPA проще интерпретируется, но менее теоретически универсален.

**10. Практические области применения**

- Оцифровка музейных объектов и скульптур.
- Реверс-инжиниринг механических деталей.
- Подготовка сеток для визуализации и анализа формы.
- Предобработка данных для CAD/CAM и аддитивного производства.

**11. Заключение**

- BPA остаётся важным классическим методом реконструкции поверхности.
- Его сильная сторона — локальное, геометрически понятное построение сетки.
- Наилучшие результаты достигаются при качественном облаке точек и аккуратном подборе радиусов.

### Список рекомендованных источников

1. Bernardini, F., Mittleman, J., Rushmeier, H., Silva, C., Taubin, G. *The Ball-Pivoting Algorithm for Surface Reconstruction*. IEEE Transactions on Visualization and Computer Graphics, 1999. DOI: `10.1109/2945.817351`.
2. Open3D Documentation. *Surface Reconstruction*. Раздел с реализацией `create_from_point_cloud_ball_pivoting`: https://www.open3d.org/docs/latest/tutorial/Advanced/surface_reconstruction.html
3. Berger, M., et al. *A Survey of Surface Reconstruction from Point Clouds*. Computer Graphics Forum, 2017.
4. Kazhdan, M., Bolitho, M., Hoppe, H. *Poisson Surface Reconstruction*. Symposium on Geometry Processing, 2006.
5. Botsch, M., Kobbelt, L., Pauly, M., Alliez, P., Levy, B. *Polygon Mesh Processing*. A K Peters, 2010.

### Практическая демонстрация или эксперимент в Google Colab

Уместный Colab-эксперимент:

1. Загрузить или сгенерировать облако точек поверхности.
2. Оценить нормали.
3. Запустить `open3d.geometry.TriangleMesh.create_from_point_cloud_ball_pivoting(...)`.
4. Повторить реконструкцию для нескольких наборов радиусов.
5. Сравнить число треугольников, число дыр и визуальное качество поверхности.

Минимальный сценарий:

```python
!pip -q install open3d

import open3d as o3d
import numpy as np

mesh_data = o3d.data.BunnyMesh()
mesh = o3d.io.read_triangle_mesh(mesh_data.path)
pcd = mesh.sample_points_poisson_disk(3000)
pcd.estimate_normals()

radii = o3d.utility.DoubleVector([0.005, 0.01, 0.02, 0.04])
bpa_mesh = o3d.geometry.TriangleMesh.create_from_point_cloud_ball_pivoting(pcd, radii)

print("Число вершин:", np.asarray(bpa_mesh.vertices).shape[0])
print("Число треугольников:", np.asarray(bpa_mesh.triangles).shape[0])
o3d.io.write_triangle_mesh("bpa_bunny.ply", bpa_mesh)
```

## Задание 7. Реферат по методу Delaunay Triangulation (3D)

### Исходная формулировка

> Подготовить реферат на тему «Поверхностная реконструкция на основе метода «Delaunay Triangulation (3D)».

### Подробный план реферата

1. Введение.
2. Базовые понятия: триангуляция Делоне и диаграмма Вороного.
3. Переход от 2D к 3D: тетраэдризация Делоне.
4. Как из 3D Delaunay-структуры получают поверхность.
5. Связь Delaunay-подходов с `alpha shapes`, `crust`, `restricted Delaunay triangulation`.
6. Основные этапы алгоритмической обработки.
7. Метрики качества реконструированной поверхности.
8. Сильные и слабые стороны подхода.
9. Сравнение с BPA и Poisson-реконструкцией.
10. Практические применения.
11. Заключение.

### Ключевые тезисы по каждому разделу

**1. Введение**

- Delaunay-подходы занимают центральное место в вычислительной геометрии и реконструкции поверхностей.
- Их ценность в том, что они связывают геометрию точек, топологию соседства и формальные гарантии качества.
- В 3D речь идёт не о треугольниках плоскости, а о тетраэдрическом разбиении пространства.

**2. Триангуляция Делоне и диаграмма Вороного**

- Триангуляция Делоне — дуальная структура к диаграмме Вороного.
- В 2D и 3D она стремится избегать вырожденных элементов и хорошо отражает локальное соседство точек.
- Ключевое свойство: пустая описанная сфера для симплексов Delaunay.

**3. Переход к 3D**

- В 3D строится тетраэдризация Делоне по набору точек.
- Поверхность объекта не задана явно, поэтому задача состоит в выделении правильной подструктуры из полного 3D-разбиения.
- Это усложняет алгоритм по сравнению с 2D-задачами.

**4. Извлечение поверхности**

- Сама по себе Delaunay-тетраэдризация — это объёмная структура, а не поверхность.
- Поверхность извлекают через специальные критерии: `alpha shapes`, `crust`, `power crust`, `restricted Delaunay`.
- Общая идея — выбрать те грани тетраэдров, которые лучше всего аппроксимируют реальную границу объекта.

**5. Связанные методы**

- `Alpha shapes` позволяют регулировать уровень детализации параметром `alpha`.
- `Crust` и `power crust` ориентированы на более теоретически обоснованное восстановление поверхности.
- `Restricted Delaunay triangulation` особенно важна в современных алгоритмах генерации и восстановления поверхностных сеток.

**6. Этапы алгоритмической обработки**

- Построение Delaunay-тетраэдризации.
- Анализ локальной геометрии и соседства.
- Фильтрация симплексов или граней по геометрическому критерию.
- Извлечение внешней оболочки.
- Постобработка: удаление шума, исправление топологии, оптимизация сетки.

**7. Метрики качества**

- Геометрическая точность: расстояния от точек до поверхности и наоборот.
- Топологическая корректность: число компонент связности, наличие дыр, самопересечений.
- Качество элементов сетки: углы, отношение сторон, устойчивость к вырожденности.

**8. Преимущества и недостатки**

- Плюсы: сильная теоретическая база, связь с вычислительной геометрией, богатый набор вариантов алгоритмов.
- Минусы: чувствительность к шуму, сложность реализации и более тяжёлые вычисления по сравнению с простыми локальными схемами.
- Для очень шумных или неполных данных может понадобиться предварительная фильтрация и регуляризация.

**9. Сравнение с BPA и Poisson**

- BPA проще объяснить геометрически и часто проще применять к чистым данным с нормалями.
- Delaunay-подходы чаще дают более формализуемую геометрическую структуру и лучше стыкуются с теориями реконструкции.
- Poisson хорошо работает для гладких закрытых поверхностей, но обычно менее интерполирующий по отношению к исходным точкам.

**10. Практические применения**

- Реконструкция анатомических поверхностей по медицинским данным.
- Геометрическая обработка сканов для обратного проектирования.
- Генерация поверхностей и объёмных сеток для численного моделирования.
- Научная визуализация распределений точек и экспериментальных измерений.

**11. Заключение**

- Методы на основе Delaunay Triangulation являются фундаментом большого класса алгоритмов реконструкции.
- Они особенно полезны там, где важны геометрическая строгость и управляемость качества сетки.
- На практике их часто используют вместе с фильтрацией, оценкой нормалей и последующей оптимизацией поверхности.

### Список рекомендованных источников

1. Edelsbrunner, H., Mücke, E. P. *Three-Dimensional Alpha Shapes*. ACM Transactions on Graphics, 1994. DOI: `10.1145/147130.147153`.
2. Amenta, N., Bern, M., Kamvysselis, M. *A New Voronoi-Based Surface Reconstruction Algorithm*. Proceedings of SIGGRAPH, 1998.
3. CGAL Documentation. *3D Mesh Generation* и *3D Surface Mesh Generation*: https://doc.cgal.org/latest/Mesh_3/ и https://doc.cgal.org/latest/Surface_mesher/
4. Oudot, S. *Geometry Processing with Delaunay Triangulations*. SIGGRAPH Course Notes / учебные материалы по вычислительной геометрии.
5. Berger, M., et al. *A Survey of Surface Reconstruction from Point Clouds*. Computer Graphics Forum, 2017.
6. Boissonnat, J.-D., Oudot, S. Исследования по `restricted Delaunay triangulation` и геометрическим гарантиям поверхностной реконструкции.

### Практическая демонстрация или эксперимент в Google Colab

Для Colab уместно показать Delaunay-связанный эксперимент через `alpha shape`, поскольку этот метод непосредственно опирается на 3D Delaunay-тетраэдризацию.

Возможный план эксперимента:

1. Сгенерировать облако точек сферы или взять небольшой реальный скан.
2. Построить `alpha shape` при нескольких значениях `alpha`.
3. Сравнить:
   - полноту поверхности;
   - число компонент связности;
   - число треугольников;
   - чувствительность к шуму.

Минимальный сценарий:

```python
!pip -q install open3d

import open3d as o3d
import numpy as np

mesh_data = o3d.data.BunnyMesh()
mesh = o3d.io.read_triangle_mesh(mesh_data.path)
pcd = mesh.sample_points_poisson_disk(3000)

alpha = 0.03
alpha_mesh = o3d.geometry.TriangleMesh.create_from_point_cloud_alpha_shape(pcd, alpha)
alpha_mesh.compute_vertex_normals()

print("Число вершин:", np.asarray(alpha_mesh.vertices).shape[0])
print("Число треугольников:", np.asarray(alpha_mesh.triangles).shape[0])
o3d.io.write_triangle_mesh("alpha_shape_bunny.ply", alpha_mesh)
```

## Итоговое замечание

Во всех практических заданиях код специально сделан учебным и автономным, чтобы его можно было запускать в Google Colab без внешней ручной подготовки. Для перехода к реальным данным достаточно заменить синтетическую генерацию на чтение `PLY`, `PCD`, `LAS`, `LAZ` или собственного формата и при необходимости усилить модели, метрики и этапы предобработки.
