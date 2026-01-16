# Exercise 06: Geometry Algorithms

## Concepts Covered
- **1.7.12.d-l** Computational geometry, convex hull, line intersection
- **1.7.13.d-k** Voronoi diagrams, Delaunay triangulation

## Objective

Implement fundamental computational geometry algorithms.

## Requirements

### Rust Implementation

```rust
pub mod primitives {
    use std::cmp::Ordering;

    #[derive(Clone, Copy, Debug, PartialEq)]
    pub struct Point {
        pub x: f64,
        pub y: f64,
    }

    impl Point {
        pub fn new(x: f64, y: f64) -> Self;
        pub fn distance(&self, other: &Point) -> f64;
        pub fn dot(&self, other: &Point) -> f64;
        pub fn cross(&self, other: &Point) -> f64;
        pub fn magnitude(&self) -> f64;
        pub fn normalize(&self) -> Point;
        pub fn rotate(&self, angle: f64) -> Point;
    }

    #[derive(Clone, Copy, Debug)]
    pub struct Line {
        pub a: f64,  // ax + by + c = 0
        pub b: f64,
        pub c: f64,
    }

    impl Line {
        pub fn from_points(p1: &Point, p2: &Point) -> Self;
        pub fn from_point_direction(p: &Point, dir: &Point) -> Self;
        pub fn distance_to_point(&self, p: &Point) -> f64;
        pub fn is_parallel(&self, other: &Line) -> bool;
    }

    #[derive(Clone, Copy, Debug)]
    pub struct Segment {
        pub p1: Point,
        pub p2: Point,
    }

    impl Segment {
        pub fn new(p1: Point, p2: Point) -> Self;
        pub fn length(&self) -> f64;
        pub fn midpoint(&self) -> Point;
        pub fn contains_point(&self, p: &Point) -> bool;
    }

    /// Cross product of (o->a) and (o->b)
    pub fn cross(o: &Point, a: &Point, b: &Point) -> f64;

    /// Orientation: -1 (CW), 0 (collinear), 1 (CCW)
    pub fn orientation(p: &Point, q: &Point, r: &Point) -> i32;

    /// Check if point is inside triangle
    pub fn point_in_triangle(p: &Point, t: &[Point; 3]) -> bool;

    /// Check if segments intersect
    pub fn segments_intersect(s1: &Segment, s2: &Segment) -> bool;

    /// Find intersection point of two lines
    pub fn line_intersection(l1: &Line, l2: &Line) -> Option<Point>;

    /// Find intersection point of two segments
    pub fn segment_intersection(s1: &Segment, s2: &Segment) -> Option<Point>;
}

pub mod convex_hull {
    use super::primitives::Point;

    /// Convex hull using Graham scan - O(n log n)
    pub fn graham_scan(points: &[Point]) -> Vec<Point>;

    /// Convex hull using Andrew's monotone chain - O(n log n)
    pub fn andrew_chain(points: &[Point]) -> Vec<Point>;

    /// Jarvis march (gift wrapping) - O(nh)
    pub fn jarvis_march(points: &[Point]) -> Vec<Point>;

    /// Check if point is inside convex polygon - O(log n)
    pub fn point_in_convex_polygon(p: &Point, hull: &[Point]) -> bool;

    /// Convex hull diameter (rotating calipers)
    pub fn convex_diameter(hull: &[Point]) -> f64;

    /// Minimum enclosing rectangle
    pub fn min_enclosing_rectangle(hull: &[Point]) -> (f64, [Point; 4]);

    /// Convex hull intersection
    pub fn hull_intersection(h1: &[Point], h2: &[Point]) -> Vec<Point>;
}

pub mod polygon {
    use super::primitives::Point;

    /// Area of simple polygon (signed)
    pub fn polygon_area(vertices: &[Point]) -> f64;

    /// Centroid of polygon
    pub fn polygon_centroid(vertices: &[Point]) -> Point;

    /// Check if polygon is convex
    pub fn is_convex(vertices: &[Point]) -> bool;

    /// Point in polygon (ray casting)
    pub fn point_in_polygon(p: &Point, polygon: &[Point]) -> bool;

    /// Polygon triangulation (ear clipping)
    pub fn triangulate(polygon: &[Point]) -> Vec<[usize; 3]>;

    /// Minimum enclosing circle
    pub fn min_enclosing_circle(points: &[Point]) -> (Point, f64);

    /// Convex polygon cut by line
    pub fn polygon_cut(polygon: &[Point], line: &super::primitives::Line) -> Vec<Point>;
}

pub mod line_sweep {
    use super::primitives::{Point, Segment};

    /// Find all intersection points among segments
    pub fn bentley_ottmann(segments: &[Segment]) -> Vec<Point>;

    /// Closest pair of points - O(n log n)
    pub fn closest_pair(points: &[Point]) -> (Point, Point, f64);

    /// Farthest pair of points
    pub fn farthest_pair(points: &[Point]) -> (Point, Point, f64);

    /// Rectangle union area
    pub fn rectangle_union_area(rects: &[(f64, f64, f64, f64)]) -> f64;

    /// Skyline problem
    pub fn skyline(buildings: &[(i32, i32, i32)]) -> Vec<(i32, i32)>;
}

pub mod voronoi_delaunay {
    use super::primitives::Point;

    /// Delaunay triangulation
    pub fn delaunay(points: &[Point]) -> Vec<[usize; 3]>;

    /// Voronoi diagram (dual of Delaunay)
    pub fn voronoi(points: &[Point]) -> Vec<Vec<Point>>;

    /// Fortune's algorithm for Voronoi - O(n log n)
    pub fn fortune_voronoi(points: &[Point]) -> VoronoiDiagram;

    pub struct VoronoiDiagram {
        pub cells: Vec<VoronoiCell>,
        pub vertices: Vec<Point>,
        pub edges: Vec<VoronoiEdge>,
    }

    pub struct VoronoiCell {
        pub site: Point,
        pub edges: Vec<usize>,
    }

    pub struct VoronoiEdge {
        pub start: Option<usize>,
        pub end: Option<usize>,
        pub left_cell: usize,
        pub right_cell: usize,
    }
}

pub mod misc {
    use super::primitives::Point;

    /// Half-plane intersection
    pub fn half_plane_intersection(planes: &[(f64, f64, f64)]) -> Option<Vec<Point>>;

    /// Rotating calipers applications
    pub fn rotating_calipers<F, T>(hull: &[Point], f: F) -> T
    where
        F: Fn(&Point, &Point, &Point, &Point) -> T;

    /// Point location in planar subdivision
    pub struct PointLocation {
        // Kirkpatrick's algorithm or trapezoidal map
    }

    impl PointLocation {
        pub fn new(triangles: &[[Point; 3]]) -> Self;
        pub fn locate(&self, p: &Point) -> Option<usize>;
    }

    /// 3D geometry: plane from three points
    pub fn plane_from_points(p1: &Point3D, p2: &Point3D, p3: &Point3D) -> Plane;

    #[derive(Clone, Copy, Debug)]
    pub struct Point3D {
        pub x: f64,
        pub y: f64,
        pub z: f64,
    }

    #[derive(Clone, Copy, Debug)]
    pub struct Plane {
        pub a: f64,
        pub b: f64,
        pub c: f64,
        pub d: f64,
    }
}
```

### Python Implementation

```python
from typing import List, Tuple, Optional
from dataclasses import dataclass
import math

@dataclass
class Point:
    x: float
    y: float

    def distance(self, other: 'Point') -> float: ...
    def cross(self, other: 'Point') -> float: ...

def cross(o: Point, a: Point, b: Point) -> float: ...
def orientation(p: Point, q: Point, r: Point) -> int: ...

def graham_scan(points: List[Point]) -> List[Point]: ...
def andrew_chain(points: List[Point]) -> List[Point]: ...

def polygon_area(vertices: List[Point]) -> float: ...
def point_in_polygon(p: Point, polygon: List[Point]) -> bool: ...

def closest_pair(points: List[Point]) -> Tuple[Point, Point, float]: ...
def min_enclosing_circle(points: List[Point]) -> Tuple[Point, float]: ...

def delaunay(points: List[Point]) -> List[Tuple[int, int, int]]: ...
```

## Test Cases

```rust
#[test]
fn test_cross_product() {
    let o = Point::new(0.0, 0.0);
    let a = Point::new(1.0, 0.0);
    let b = Point::new(0.0, 1.0);

    assert!((cross(&o, &a, &b) - 1.0).abs() < 1e-9);  // CCW
    assert!((cross(&o, &b, &a) - (-1.0)).abs() < 1e-9);  // CW
}

#[test]
fn test_orientation() {
    let p = Point::new(0.0, 0.0);
    let q = Point::new(1.0, 1.0);
    let r = Point::new(2.0, 2.0);

    assert_eq!(orientation(&p, &q, &r), 0);  // Collinear

    let r2 = Point::new(2.0, 0.0);
    assert_eq!(orientation(&p, &q, &r2), -1);  // CW
}

#[test]
fn test_convex_hull() {
    let points = vec![
        Point::new(0.0, 0.0),
        Point::new(1.0, 1.0),
        Point::new(2.0, 0.0),
        Point::new(1.0, 2.0),
        Point::new(1.0, 0.5),  // Interior point
    ];

    let hull = graham_scan(&points);
    assert_eq!(hull.len(), 4);  // Square without interior point
}

#[test]
fn test_polygon_area() {
    let square = vec![
        Point::new(0.0, 0.0),
        Point::new(2.0, 0.0),
        Point::new(2.0, 2.0),
        Point::new(0.0, 2.0),
    ];
    assert!((polygon_area(&square).abs() - 4.0).abs() < 1e-9);
}

#[test]
fn test_point_in_polygon() {
    let polygon = vec![
        Point::new(0.0, 0.0),
        Point::new(4.0, 0.0),
        Point::new(4.0, 4.0),
        Point::new(0.0, 4.0),
    ];

    assert!(point_in_polygon(&Point::new(2.0, 2.0), &polygon));
    assert!(!point_in_polygon(&Point::new(5.0, 2.0), &polygon));
}

#[test]
fn test_segment_intersection() {
    let s1 = Segment::new(Point::new(0.0, 0.0), Point::new(2.0, 2.0));
    let s2 = Segment::new(Point::new(0.0, 2.0), Point::new(2.0, 0.0));

    assert!(segments_intersect(&s1, &s2));

    let intersection = segment_intersection(&s1, &s2).unwrap();
    assert!((intersection.x - 1.0).abs() < 1e-9);
    assert!((intersection.y - 1.0).abs() < 1e-9);
}

#[test]
fn test_closest_pair() {
    let points = vec![
        Point::new(0.0, 0.0),
        Point::new(1.0, 1.0),
        Point::new(3.0, 3.0),
        Point::new(0.1, 0.0),
    ];

    let (p1, p2, dist) = closest_pair(&points);
    assert!((dist - 0.1).abs() < 1e-9);
}

#[test]
fn test_min_enclosing_circle() {
    let points = vec![
        Point::new(0.0, 0.0),
        Point::new(2.0, 0.0),
        Point::new(1.0, 1.0),
    ];

    let (center, radius) = min_enclosing_circle(&points);

    // All points should be inside or on circle
    for p in &points {
        assert!(center.distance(p) <= radius + 1e-9);
    }
}

#[test]
fn test_delaunay() {
    let points = vec![
        Point::new(0.0, 0.0),
        Point::new(1.0, 0.0),
        Point::new(0.5, 0.866),
        Point::new(0.5, 0.289),  // Center of equilateral triangle
    ];

    let triangles = delaunay(&points);

    // Should produce 3 triangles
    assert_eq!(triangles.len(), 3);

    // Verify Delaunay property: no point inside circumcircle
    for tri in &triangles {
        // Check circumcircle is empty
    }
}

#[test]
fn test_convex_diameter() {
    let hull = vec![
        Point::new(0.0, 0.0),
        Point::new(3.0, 0.0),
        Point::new(3.0, 4.0),
        Point::new(0.0, 4.0),
    ];

    let diameter = convex_diameter(&hull);
    assert!((diameter - 5.0).abs() < 1e-9);  // Diagonal of 3x4 rectangle
}

#[test]
fn test_skyline() {
    let buildings = vec![
        (2, 9, 10),
        (3, 7, 15),
        (5, 12, 12),
        (15, 20, 10),
        (19, 24, 8),
    ];

    let skyline = skyline(&buildings);
    // Expected: [(2,10), (3,15), (7,12), (12,0), (15,10), (20,8), (24,0)]
}

#[test]
fn test_rectangle_union_area() {
    let rects = vec![
        (0.0, 0.0, 2.0, 2.0),
        (1.0, 1.0, 3.0, 3.0),
    ];

    let area = rectangle_union_area(&rects);
    assert!((area - 7.0).abs() < 1e-9);  // 4 + 4 - 1 overlap = 7
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Primitives & orientation | 10 |
| Convex hull | 20 |
| Polygon operations | 15 |
| Line sweep algorithms | 20 |
| Delaunay/Voronoi | 20 |
| Min enclosing circle | 10 |
| Edge cases | 5 |
| **Total** | **100** |
