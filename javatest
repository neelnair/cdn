"use strict";

const svg = {
	elem: document.querySelector("svg"),
	group: document.createElementNS("http://www.w3.org/2000/svg", "g"),
	width: 200,
	height: 200,
	complete: false,

	init() {
		this.elem.setAttribute("viewBox", "0 0 200 200");
		this.elem.setAttribute("width", `${this.width}mm`);
		this.elem.setAttribute("height", `${this.height}mm`);
		this.elem.setAttribute("style", "background:#fcfbfa");
		this.elem.setAttribute("viewport-fill", "#fcfbfa");

		this.group.setAttribute("stroke-linejoin", "round");
		this.group.setAttribute(
			"style",
			"fill:none;stroke:#222;stroke-width:0.3;opacity:1"
		);

		this.elem.addEventListener(
			"click",
			(e) => {
				if (this.complete) this.saveSVGFile();
			},
			false
		);
		this.elem.appendChild(this.group);
	},

	insertPolyline(points) {
		const clippedPaths = this.clip(points, 0, this.width);
		for (const points of clippedPaths) {
			const poly = document.createElementNS(
				"http://www.w3.org/2000/svg",
				"polyline"
			);
			const svgPoints = points.map((n) => +n.toFixed(2));
			poly.setAttribute("points", svgPoints.toString());
			this.group.appendChild(poly);
		}
	},

	requestAnimationFrame() {
		return new Promise((resolve) => window.requestAnimationFrame(resolve));
	},

	optimizePath: async function (lines, penWidth) {
		// based on https://github.com/Evelios/optimize-path
		let freq = location.pathname.match(/fullcpgrid/i) === null ? 10 : 1000;
		let frm = 0;
		if (lines.length === 0) return [];
		const pws = penWidth ** 2;
		let frontier = lines.slice(0);
		let cNode = frontier.pop();
		let combined = cNode;
		while (frontier.length !== 0) {
			let reversed = false;
			let pathIndex = -1;
			let closestDist = Infinity;
			let dist = Infinity;
			let cEndX = cNode[cNode.length - 2];
			let cEndY = cNode[cNode.length - 1];
			// Get the path that is closest to the current node
			for (let index = 0; index < frontier.length; index++) {
				const path = frontier[index];
				// Regular Orientation
				dist = (cEndX - path[0]) ** 2 + (cEndY - path[1]) ** 2;
				if (dist < closestDist) {
					reversed = false;
					pathIndex = index;
					closestDist = dist;
				}
				// Reversed Orientation
				dist =
					(cEndX - path[path.length - 2]) ** 2 +
					(cEndY - path[path.length - 1]) ** 2;
				if (dist < closestDist) {
					reversed = true;
					pathIndex = index;
					closestDist = dist;
				}
			}
			// Add the closest path to the explored list and remove it from the frontier
			cNode = frontier[pathIndex];
			frontier.splice(pathIndex, 1);
			if (reversed) {
				const new_node = [];
				for (let i = cNode.length - 2; i >= 0; i -= 2) {
					new_node.push(cNode[i]);
					new_node.push(cNode[i + 1]);
				}
				cNode = new_node;
			}
			// If the paths are closer than the pen width, them combine them
			if (closestDist < pws) {
				combined = combined.concat(cNode);
			} else {
				if (combined.length > 0) {
					this.insertPolyline(combined);
					combined = [];
				}
				combined = cNode;
				this.insertPolyline(cNode);
				if (++frm % freq === 0) await this.requestAnimationFrame();
			}
		}
		this.complete = true;
	},

	// port of http://paulbourke.net/geometry/pointlineplane/Helpers.cs
	intersect(pint, a, b, c, d) {
		const e = (d[1] - c[1]) * (b[0] - a[0]) - (d[0] - c[0]) * (b[1] - a[1]);
		if (e === 0) return false;
		const ua =
			((d[0] - c[0]) * (a[1] - c[1]) - (d[1] - c[1]) * (a[0] - c[0])) / e;
		const ub =
			((b[0] - a[0]) * (a[1] - c[1]) - (b[1] - a[1]) * (a[0] - c[0])) / e;
		if (ua >= 0 && ua <= 1 && ub >= 0 && ub <= 1) {
			pint[0] = a[0] + ua * (b[0] - a[0]);
			pint[1] = a[1] + ua * (b[1] - a[1]);
			return true;
		}
		return false;
	},

	// adapted from https://turtletoy.net/js/turtlesvg.js
	clip(polyline, left, size) {
		let pint = [0, 0];
		const clip = [left, left, left, size, size, size, size, left, left, left];
		const nps = [];
		let np = [];
		let pcx = polyline[0];
		let pcy = polyline[1];
		let inside = pcx > left && pcx < size && pcy > left && pcy < size;
		if (inside === true) np.push(pcx, pcy);
		for (let j = 0; j < polyline.length; j += 2) {
			const cx = polyline[j];
			const cy = polyline[j + 1];
			if (cx === pcx && cy === pcy && j < polyline.length - 2) continue;
			if (cx > left && cx < size && cy > left && cy < size) {
				if (inside) np.push(cx, cy);
				else {
					for (let i = 0; i < 8; i += 2) {
						if (
							this.intersect(
								pint,
								[pcx, pcy],
								[cx, cy],
								[clip[i], clip[i + 1]],
								[clip[i + 2], clip[i + 3]]
							) === true
						)
							break;
					}
					np.push(pint[0], pint[1], cx, cy);
				}
				inside = true;
			} else {
				if (inside) {
					for (let i = 0; i < 8; i += 2) {
						if (
							this.intersect(
								pint,
								[pcx, pcy],
								[cx, cy],
								[clip[i], clip[i + 1]],
								[clip[i + 2], clip[i + 3]]
							) === true
						)
							break;
					}
					np.push(pint[0], pint[1]);
					nps.push(np);
					np = [];
				} else {
					const ips = [];
					for (let i = 0; i < 8; i += 2) {
						if (
							this.intersect(
								pint,
								[pcx, pcy],
								[cx, cy],
								[clip[i], clip[i + 1]],
								[clip[i + 2], clip[i + 3]]
							) === true
						)
							ips.push(pint[0], pint[1]);
					}
					if (ips.length === 4) nps.push(ips);
				}
				inside = false;
			}
			pcx = cx;
			pcy = cy;
		}
		if (np.length > 0) nps.push(np);
		return nps;
	},

	saveSVGFile() {
		this.elem.setAttribute("xmlns", "http://www.w3.org/2000/svg");
		const svgData = this.elem.outerHTML;
		const preface = '<?xml version="1.0" encoding="UTF-8" standalone="no"?>\r\n';
		const svgBlob = new Blob([preface, svgData], {
			type: "image/svg+xml;charset=utf-8"
		});
		const svgUrl = URL.createObjectURL(svgBlob);
		const downloadLink = document.createElement("a");
		downloadLink.href = svgUrl;
		downloadLink.download = "codepen.svg";
		document.body.appendChild(downloadLink);
		downloadLink.click();
		document.body.removeChild(downloadLink);
	}
};

const voronoi = {
	/**
	 * Ported from "Sketch of Voronoi"
	 * Dead Code Preservation: http://wa.zozuar.org/code.php?c=iNy0
	 * @see http://en.wikipedia.org/wiki/Fortune's_algorithm
	 * C++ reference https://www.cs.hmc.edu/~mbrubeck/voronoi.html
	 */

	polylines: [],

	Point: class Point {
		constructor(x = 0.0, y = 0.0) {
			this.x = x;
			this.y = y;
		}
	},

	Arc: class Arc {
		constructor(p, prev, next) {
			this.p = p;
			this.next = next;
			this.prev = prev;
			this.v0 = null;
			this.v1 = null;
			this.left = null;
			this.right = null;
			this.endP = null;
			this.endX = 0.0;
		}
	},

	intersection(p0, p1, l, res) {
		let p = p0,
			ll = l * l;
		if (p0.x === p1.x) res.y = (p0.y + p1.y) / 2;
		else if (p1.x === l) res.y = p1.y;
		else if (p0.x === l) {
			res.y = p0.y;
			p = p1;
		} else {
			const z0 = 0.5 / (p0.x - l);
			const z1 = 0.5 / (p1.x - l);
			const a = z0 - z1;
			const b = -2 * (p0.y * z0 - p1.y * z1);
			const c =
				(p0.y * p0.y + p0.x * p0.x - ll) * z0 -
				(p1.y * p1.y + p1.x * p1.x - ll) * z1;
			res.y = (-b - Math.sqrt(b * b - 4 * a * c)) / (2 * a);
		}
		res.x = (p.x * p.x + (p.y - res.y) * (p.y - res.y) - ll) / (2 * p.x - 2 * l);
	},

	fortune(points) {
		const N = points.length;
		let o = new this.Point();
		let root = null;
		let a = null;
		let b = null;
		let c = null;
		let d = null;
		let next = null;
		let eventX = 0;
		let w = points[0].x;
		for (let i = 1; i < N; i++) {
			const p = points[i];
			const x = p.x;
			if (x < w) {
				let j = i;
				while (j > 0 && points[j - 1].x > x) {
					points[j] = points[j - 1];
					j--;
				}
				points[j] = p;
			} else w = x;
		}
		const x0 = points[0].x;
		let i = 0;
		let p = points[0];
		let x = p.x;
		while (true) {
			if (a !== null) {
				let circle = false;
				if (a.prev !== null && a.next !== null) {
					const aa = a.prev.p;
					const bb = a.p;
					const cc = a.next.p;
					let A = bb.x - aa.x;
					let B = bb.y - aa.y;
					const C = cc.x - aa.x;
					const D = cc.y - aa.y;
					if (A * D - C * B <= 0) {
						const E = A * (aa.x + bb.x) + B * (aa.y + bb.y);
						const F = C * (aa.x + cc.x) + D * (aa.y + cc.y);
						const G = 2 * (A * (cc.y - bb.y) - B * (cc.x - bb.x));
						if (G !== 0) {
							o.x = (D * E - B * F) / G;
							o.y = (A * F - C * E) / G;
							A = aa.x - o.x;
							B = aa.y - o.y;
							eventX = o.x + Math.sqrt(A * A + B * B);
							if (eventX >= w) circle = true;
						}
					}
				}
				if (a.right !== null) a.right.left = a.left;
				if (a.left !== null) a.left.right = a.right;
				if (a === next) next = a.right;
				if (circle === true) {
					a.endX = eventX;
					if (a.endP !== null) {
						a.endP.x = o.x;
						a.endP.y = o.y;
					} else {
						a.endP = o;
						o = new this.Point();
					}
					d = next;
					if (d === null) {
						next = a;
					} else
						while (true) {
							if (d.endX >= eventX) {
								a.left = d.left;
								if (d.left !== null) d.left.right = a;
								if (next === d) next = a;
								a.right = d;
								d.left = a;
								break;
							}
							if (d.right === null) {
								d.right = a;
								a.left = d;
								a.right = null;
								break;
							}
							d = d.right;
						}
				}
				if (b !== null) {
					a = b;
					b = null;
					continue;
				}
				if (c !== null) {
					a = c;
					c = null;
					continue;
				}
				a = null;
			}
			if (next !== null && next.endX <= x) {
				a = next;
				next = a.right;
				if (next !== null) next.left = null;
				a.right = null;
				if (a.prev !== null) {
					a.prev.next = a.next;
					a.prev.v1 = a.endP;
				}
				if (a.next !== null) {
					a.next.prev = a.prev;
					a.next.v0 = a.endP;
				}
				this.polylines.push([a.v0.x, a.v0.y, a.endP.x, a.endP.y, a.v1.x, a.v1.y]);
				d = a;
				w = a.endX;
				if (a.prev !== null) {
					b = a.prev;
					a = a.next;
				} else {
					a = a.next;
					b = null;
				}
			} else {
				if (p === null) break;
				if (root === null) {
					root = new this.Arc(p, null, null);
				} else {
					let z = new this.Point();
					a = root.next;
					if (a !== null) {
						while (a.next !== null) {
							a = a.next;
							if (a.p.y >= p.y) break;
						}
						this.intersection(a.prev.p, a.p, p.x, z);
						if (z.y <= p.y) {
							while (a.next !== null) {
								a = a.next;
								this.intersection(a.prev.p, a.p, p.x, z);
								if (z.y >= p.y) {
									a = a.prev;
									break;
								}
							}
						} else {
							a = a.prev;
							while (a.prev !== null) {
								a = a.prev;
								this.intersection(a.p, a.next.p, p.x, z);
								if (z.y <= p.y) {
									a = a.next;
									break;
								}
							}
						}
					} else a = root;
					if (a.next !== null) {
						b = new this.Arc(a.p, a, a.next);
						a.next.prev = b;
						a.next = b;
					} else {
						b = new this.Arc(a.p, a, null);
						a.next = b;
					}
					a.next.v1 = a.v1;
					z.y = p.y;
					z.x =
						(a.p.x * a.p.x + (a.p.y - p.y) * (a.p.y - p.y) - p.x * p.x) /
						(2 * a.p.x - 2 * p.x);
					b = new this.Arc(p, a, a.next);
					a.next.prev = b;
					a.next = b;
					a = a.next;
					a.prev.v1 = z;
					a.next.v0 = z;
					a.v0 = z;
					a.v1 = z;
					b = a.next;
					a = a.prev;
					c = null;
					w = p.x;
				}
				i++;
				if (i >= N) {
					p = null;
					x = 999999;
				} else {
					p = points[i];
					x = p.x;
				}
			}
		}
	}
};

const img = {
	img: new Image(),
	points: [],
	////////////////////
	zoom: 4,
	contrast: 128,
	////////////////////

	decode() {
		const w = this.img.width * this.zoom;
		const h = this.img.height * this.zoom;
		const cmap = document.createElement("canvas");
		cmap.width = w;
		cmap.height = h;
		const ctx = cmap.getContext("2d");
		ctx.drawImage(this.img, 0, 0, w, h);
		const data = ctx.getImageData(0, 0, w, h).data;
		const rx = svg.width / w;
		const ry = svg.height / h;
		// decode brillance
		for (let i = 0; i < w * h * 4; i += 4) {
			const bri = 0.34 * data[i] + 0.5 * data[i + 1] + 0.16 * data[i + 2];
			if (Math.random() > bri / this.contrast) {
				this.points.push(
					new voronoi.Point(
						Math.random() + ((i / 4) % w) * rx,
						Math.random() + (i / 4 / w) * ry
					)
				);
			}
		}
	},
	load(id) {
		return new Promise((resolve) => {
			this.img.crossOrigin = "Anonymous";
			this.img.addEventListener("load", (_) => resolve());
			this.img.src = document.getElementById(id).src;
		});
	}
};

const run = async () => {
	await img.load("source");
	svg.init();
	img.decode();
	voronoi.fortune(img.points);
	svg.optimizePath(voronoi.polylines, 0.3);
};

run();
