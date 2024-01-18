---
Title: Front War 2 - When the lead strikes the steel - C#/Javascript/traçabilité
Subtitle: ""
Date: 2023-12-14
Lastmod : 
Tags: ["Prog"]
Description: "Article sur le dernier projet personnel."
Software: Unity 3D
---
![Alt text](/readme_files/pikes_shots.jpg "")

# Front_war_2
Novembre 2023 - When the lead strikes the steel - Jeu RTS Dans la nostalgie et dans la volonté de continuer ma passion, je décide de reprendre les idées de mon premier projet de Jeu datant de 2016. - Jeu vidéo
Objectif open source 

J'expliquerai ici comment : 
1. Télécharger et jouer la dernière version accessible 
2. Télécharger et modifier sur unity la dernière version accessible 
3. Utiliser certains codes sans tout télécharger : Triangulation, player raycasting ,code des unités.
4. Télécharger et modifier les visuels utilisés. Tous open-sources.  


# Novembre 2023 - *When the lead strikes the steel* - Jeu RTS 

*Dans la nostalgie et dans la volonté de continuer ma passion, je décide de reprendre les idées de mon premier projet de Jeu datant de 2016. - Jeu vidéo*

*Je souhaite suivre de bout en bout une rigueur de réalisation, que j'ai connu dans le milieu normatif : traçabilité, standardisation et conformité. Organigramme réalisé sous Visio :*
![Alt text](/readme_files/qualification.PNG "")

Objectifs :

- Jeu en Real Time strategy macroscopique.

- Vue orthographique et design retro (inspirations Cossacks:Art of War, Age of Empire 2 et empire earth).

- Ne doit pas depasser en taille l'espace de mon disque dur (libre) : 1 Go.
  
- Créer et faire évoluer son camp à travers l'une des époques les plus révolutionnaires au niveau technique et culturel : 16ème à la Fin du 17 ème - Epoque Pikes & Shottes - L'arrivée des européens en Amérique etc.
  
- Utilisation de bibliothèques gratuites et libre de droit.
  
- Traçabilité des exigences et arborescence : réaliser l'exigence "client" jusqu'à la ligne de code.
  
- Multijoueur avec serveur client-side + ressources graphiques côté client + Pre-rendering.  
  
- Mode de jeu "scalable" en fonction du nombre de joueur. 

- Open-source et première version qui sera sur github.
  

![Alt text](/readme_files/pikes_shots_2.jpg "")


*Le principe du cycle en V sera mis en place.  Cycle en V réalisé sous Visio :*

![Alt text](/readme_files/Cycle_en_v.PNG "")

*Server client-side. Schéma à revoir, simplement illustratif :*

![Alt text](/readme_files/reseau_spec.PNG "")

*Exigence du cahier de spécification logiciel concernant les interfaces/menus :*

![Alt text](/readme_files/interfaces.PNG "")

*Exigence du cahier de spécification logiciel concernant l'architecture de programmation, on définira ici le back-end comme ce qui ne se voit pas et le front-end comme ce qui se voit :*

![Alt text](/readme_files/Organigramme_p_2_page-0001.jpg "")
![Alt text](/readme_files/Organigramme_p_2_page-0002.jpg "")
![Alt text](/readme_files/Organigramme_p_2_page-0003.jpg "")

*Civilisation Astèque, mindmap*

![Alt text](/readme_files/Aztec_tree.PNG "")


*Civilisation Mongole, mindmap*

![Alt text](/readme_files/Mughal_tree.PNG "")

*Civilisation Indienne, mindmap*

![Alt text](/readme_files/India_tree.PNG "")

*Civilisation Chinoise, mindmap*

![Alt text](/readme_files/China_tree.PNG "")

*Civilisation Française, mindmap*

![Alt text](/readme_files/France_tree.PNG "")

*Civilisation Espagnole, mindmap*

![Alt text](/readme_files/Spain_tree.PNG "")


# Code pour la triangulation de delaunay et la recherche du point le plus proche

On utilise les librairies : Triangle.NET : https://github.com/wo80/Triangle.NET/

1. Triangulation : Unity n'a pas de support intégré pour la triangulation de Delaunay. Cependant, on peut utiliser des bibliothèques tierces comme Triangle.NET qui peuvent gérer cela. La triangulation de Delaunay est un algorithme qui peut être utilisé pour générer un maillage à partir d'une liste de points Vector3. Il génère des triangles de telle sorte qu'aucun point ne se trouve à l'intérieur du cercle circonscrit d'un triangle.

2. Trouver les points les plus proches : Après la triangulation de Delaunay, on souhaitez trouver les points les plus proches dans le maillage généré. On utilise la librairie et une recherche linéaire avec un nombre de points limités. 

3. Dessiner des lignes : Le composant LineRenderer de Unity peut être utilisé pour dessiner une ligne entre deux ou plusieurs points dans l'espace 3D. Une fois que l'on a les points les plus proches, on peut utiliser LineRenderer pour dessiner des lignes entre chaque point et son voisin le plus proche.

*Résultat de triangulation des points en mouvement: *

![Alt text](/readme_files/triangulation_1.png "")

*Résultat de triangulation des points statiques: *

![Alt text](/readme_files/triangulation_2.png "")

Code fonctionnel contenant seulement la triangulation : 

```
using UnityEngine;
using TriangleNet.Meshing;
using TriangleNet.Geometry;
using TriangleNet.Meshing.Algorithm;
using TriangleNet.Meshing;
using TriangleNet.Smoothing;
using TriangleNet.IO;
using System.Collections.Generic;

public class triangulation_code : MonoBehaviour 
{
    // Unity objects.
    List<Transform> All_unit = new List<Transform>();

    // Triangle.NET vertices (class instance for better memory performance).
    List<Vertex> vertices = new List<Vertex>();

    // For better memory performance.
    TrianglePool trianglePool = new TrianglePool();
    ITriangulator triangulator = new Dwyer();

    // To find nearest neighbours.
    VertexCirculator vertexCirculator;

    LineRenderer lineRenderer;


    void Start()
    {
        // Create a LineRenderer to display on screen
        lineRenderer = gameObject.GetComponent<LineRenderer>();
        lineRenderer.widthMultiplier = 2f;
    }

    void Update()
    {
        if (All_unit == null || All_unit.Count == 0) return;

        int count = All_unit.Count;

        lineRenderer.positionCount = 0;

        vertices.Clear();

        if (vertices.Capacity < count)
        {
            vertices.EnsureCapacity(count);
        }

        // Copy the position
        for (int i = 0; i < count; i++)
        {
            var pos = All_unit[i].position;

            // IMPORTANT: set correct vertex id (corresponding to index in All_unit).
            //            This will automatically establish the mapping between Unity
            //            and Triangle.NET objects.

            vertices.Add(new Vertex(pos.x, pos.z) { ID = i });
        }

        // Perform Delaunay triangulation on vertices

        var mesh = (Mesh)triangulator.Triangulate(vertices, new Configuration(
            () => RobustPredicates.Default,
            () => trianglePool.Restart()));

        vertexCirculator = new VertexCirculator(mesh);

        // GET ALL THE Point from the triangulation in order to display it 

        var positions_upt = new List<Vector3>();

        foreach (var edge in mesh.Edges)
        {
            positions_upt.Add(All_unit[edge.P0].position);
            positions_upt.Add(All_unit[edge.P1].position);
        }

        lineRenderer.SetPositions(positions_upt.ToArray());

        // Find all nearest neighbors
        for (int i = 0; i < count; i++)
        {
            var obj = All_unit[i];
            int closedId = FindClosest(vertices[i]);

            if (closedId < 0)
            {
                // Something went wrong
            }
            else
            {
                var closest = All_unit[closedId];
            }
        }
    }

    int FindClosest(Vertex vertex)
    {
        if (vertexCirculator == null) return -1;

        Vertex closest = null;
        double closestDist = double.MaxValue;

        foreach (var neighbor in vertexCirculator.EnumerateVertices(vertex))
        {
            var dx = (vertex.X - neighbor.X);
            var dy = (vertex.Y - neighbor.Y);

            var squareDist = dx * dx + dy * dy;

            if (squareDist < closestDist)
            {
                closest = neighbor;
                closestDist = squareDist;
            }
        }

        return closest == null ? -2 : closest.ID;
    }
}

```


Logiciels / Outils : Unity 3D, Blender, mermaid, jira, confluence. 
