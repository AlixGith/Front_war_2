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

*Server client-side. Schéma à revoir, simplement illustratif :*


# Code pour la triangulation de delaunay et la recherche du point le plus proche

On utilise les librairies : UnityOctree et Triangle.NET

1. Triangulation : Unity n'a pas de support intégré pour la triangulation de Delaunay. Cependant, on peut utiliser des bibliothèques tierces comme Triangle.NET qui peuvent gérer cela. La triangulation de Delaunay est un algorithme qui peut être utilisé pour générer un maillage à partir d'une liste de points Vector3. Il génère des triangles de telle sorte qu'aucun point ne se trouve à l'intérieur du cercle circonscrit d'un triangle.

2. Trouver les points les plus proches : Après la triangulation de Delaunay, on souhaitez trouver les points les plus proches dans le maillage généré. Pour cela, on utilise la structure de données Octree dans Unity. La structure de données Octree est efficace pour les requêtes spatiales telles que la recherche des points les plus proches. 

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
    public List<Transform> All_unit = new List<Transform>();
    public List<Vector3> vertices = new List<Vector3>();
    PointOctree<Vector3> octree;

    LineRenderer lineRenderer;
    Bounds bounds; 

    void Start()
    {
        // Create a LineRenderer
            lineRenderer = gameObject.GetComponent<LineRenderer>();
            lineRenderer.widthMultiplier = 2f;
                        // Create and populate the Octree with vertices
            Bounds bounds = new Bounds(All_unit[0].position, All_unit[All_unit.Count-1].position*10); 

    }
    void Update() 
    {
        if(All_unit !=null)
        {

            lineRenderer.positionCount = 0;

            //octree = new PointOctree<Vector3>(bounds.size.x, bounds.center, 0);
            vertices.Clear(); 

            for (int i = 0; i < All_unit.Count; i++)
            {
                vertices.Add(All_unit[i].position);
                
            }


            // Perform Delaunay triangulation on vertices
            
            var input = new Polygon(All_unit.Count);
            
            
            foreach (var vertex in vertices)
            {
                input.Add(new Vertex(vertex.x, vertex.z));
                
                //octree.Add(vertex, vertex);
            }

            

            // Generate mesh.
            var options = new ConstraintOptions() { ConformingDelaunay = true};
            var quality = new QualityOptions() { MinimumAngle = 30 };
            
            var mesh = input.Triangulate(options, quality);
            
            
            // var config = new TriangleNet.Configuration();
            
            //mesh = triangulator.Triangulate(mesh,config);

            //var smoother = new SimpleSmoother();

            // Smooth mesh.
            //smoother.Smooth(mesh, All_unit.Count*2, .05);

            List<Vector3> newvertices = new List<Vector3>(); 


            foreach (var vertex in mesh.Vertices)
            {
                newvertices.Add(new Vector3((float)vertex.X, 0, (float)vertex.Y));
            }
            
           
            
            
            //lineRenderer.material = new Material(Shader.Find("Particles/Standard Unlit"));
            
            lineRenderer.positionCount = mesh.Triangles.Count*3;
            //lineRenderer.SetPositions(newvertices); 

            
            List<Vector3> positions_upt = new List<Vector3>(); 

            foreach(var triangle in mesh.Triangles)
            {
                int label = triangle.Label;

                if (label < 0 )
                {
                    Debug.Log("MFEM element attributes must be positive.");
                }else{
                    if(triangle.GetVertexID(0) < newvertices.Count && triangle.GetVertexID(0) >=0 )
                        {positions_upt.Add(newvertices[triangle.GetVertexID(0)]);}
                    if(triangle.GetVertexID(1) < newvertices.Count && triangle.GetVertexID(1) >=0 )
                        {positions_upt.Add(newvertices[triangle.GetVertexID(1)]);}
                    if(triangle.GetVertexID(2) < newvertices.Count && triangle.GetVertexID(2) >=0 )
                        {positions_upt.Add(newvertices[triangle.GetVertexID(2)]);}
                }
                
                

            }

            lineRenderer.SetPositions(positions_upt.ToArray()); 
            // Find closest points and draw lines
            /*
            for (int i = 0; i < newvertices.Count; i++)
            {
                //var closestPoint = FindNearestPoint(newvertices[i], newvertices);
                lineRenderer.SetPosition(i * 2, newvertices[i]);
                lineRenderer.SetPosition(i * 2 + 1, closestPoint);
            }
            */
           
        }
    }

    Vector3 FindNearestPoint(Vector3 point, List<Vector3> vertices)
    {
        float closestDistance = float.MaxValue;
        Vector3 closestpoint = Vector3.zero;

        foreach(var vertex in vertices)
        {
            float distance = Vector3.Distance(new Vector3(point.x,0,point.z), new Vector3(vertex.x, 0, vertex.z)); 

            if(distance < closestDistance)
            {
                closestDistance = distance; 
                closestpoint = vertex; 
            }
        }
        return closestpoint; 
    }
}

```


Logiciels / Outils : Unity 3D, Blender, mermaid, jira, confluence. 
